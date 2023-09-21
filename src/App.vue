

<template>
  <div>
    <input type="file" ref="fileInput" @change="handleFileUpload" />
    <button @click="extractText">Extract Text</button>
    <div v-if="Object.keys(cardGroups).length">
      <div v-for="cg in cardGroups">
        <div>{{ cg.groupName }}</div>
        <table>
          <thead>
            <th>消費日</th>
            <th>入帳日</th>
            <th>消費明細</th>
            <th>外幣幣別</th>
            <th>外幣金額</th>
            <th>消費金額</th>
          </thead>
          <tbody>
            <tr v-for="t in cg.transactions">
              <td>{{ t.transactionDate }}</td>
              <td>{{ t.settlementDate }}</td>
              <td>{{ t.description }}</td>
              <td>{{ t.foreignCurrency }}</td>
              <td>{{ t.foreignAmount }}</td>
              <td>{{ t.amount }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import * as pdfjsLib from 'pdfjs-dist';

// Initialize PDF.js
pdfjsLib.GlobalWorkerOptions.workerSrc = '/assets/js/pdf.worker.min.js';

const fileInput = ref<HTMLInputElement | null>(null);
const extractedText = ref<string | null>(null);

const handleFileUpload = (event: Event) => {
  const input = event.target as HTMLInputElement;
  if (input.files && input.files.length > 0) {
    fileInput.value = input;
  }
};

const rawLines = ref<string[]>([]);
interface Transaction {
  transactionDate: string | null;
  settlementDate: string | null;
  description: string;
  amount: number;
  foreignCurrency?: string;
  foreignAmount?: number;
}
interface CardGroup {
  groupName: string;
  transactions: Transaction[];
  totalAmount: number;  
}

const cardGroups = ref<Record<string, CardGroup>>({});
const transactions = ref<Transaction[]>([]);

const parseTransactions = () => {
  let isReading = false;
  let isReadingTransaction = false;
  let patternType = 0;
  let readingIndex = 0;
  let currentGroupName = '';
  let currentGroupKey = '';
  transactions.value = [];
  
  let tempTransaction: Partial<Transaction> = {};

  for (let i = 0; i < rawLines.value.length; i++) {
    let line = rawLines.value[i];

    if (line === '正卡' || line === '附卡') {
      currentGroupName = `${rawLines.value[i - 2]} ${line} ${rawLines.value[i + 2]} ${rawLines.value[i + 4]}`;
      currentGroupKey = `card_${rawLines.value[i + 4]}`;
      continue;
    }

    if (line === '您本期的消費明細如下') {
      isReading = true;
      continue;
    } else if (line === '續下頁') {
      isReading = false;
      continue;
    }

    const startPattern1 = /^(\d{4}\/\d{2}\/\d{2})\s+(\d{4}\/\d{2}\/\d{2})$/;
    const startPattern2 = /^(\d{4}\/\d{2}\/\d{2})\s+(\d{4}\/\d{2}\/\d{2})\s+([\w\s,\/\.\-\*]+)\s+(.*)$/;
    const singleDateMatch = /^(\d{4}\/\d{2}\/\d{2})$/;

    const pushTransaction = () => {
      transactions.value.push(tempTransaction as Transaction);
      if (cardGroups.value[currentGroupKey] === undefined) {
        cardGroups.value[currentGroupKey] = {
          groupName: currentGroupName,
          transactions: [],
          totalAmount: 0,
        };
      }
      const cg = cardGroups.value[currentGroupKey] as CardGroup;
      cg.transactions.push(tempTransaction as Transaction);
      cg.totalAmount += tempTransaction.amount as number;
      tempTransaction = {};
      isReadingTransaction = false;
    }

    if (isReading) {
      if (line.includes(',')) {
        line = line.replace(',', '');
      }
      if (line.match(startPattern1)) {
        patternType = 1;
        readingIndex = i;
        isReadingTransaction = true;
        let matches = line.match(startPattern1);
        tempTransaction.transactionDate = matches?.[1] ?? '';
        tempTransaction.settlementDate = matches?.[2] ?? '';
      }
      if (line.match(startPattern2)) {
        patternType = 2;
        readingIndex = i;
        isReadingTransaction = true;
        let matches = line.match(startPattern2);
        tempTransaction.transactionDate = matches?.[1] ?? '';
        tempTransaction.settlementDate = matches?.[2] ?? '';
        tempTransaction.description = line.replace(matches?.[1] ?? '', '').replace(matches?.[2] ?? '', '');
      }
      if (isReadingTransaction) {
        switch (patternType) {
          case 1:
            if (i === readingIndex + 2) {
              tempTransaction.description = line;
            } else if (i === readingIndex + 4) {
              tempTransaction.description += line;
            } else if (i > readingIndex + 5 && i < readingIndex + 12) {
              let amt = parseFloat(line);
              if (isNaN(amt)) {
                tempTransaction.description += line;
              } else {
                tempTransaction.amount = parseFloat(line);
                pushTransaction();
              }
            } 
            break;
          case 2:
            if (i === readingIndex + 2) {
              if (line.match(singleDateMatch)) {
                continue;
              } else if (line.includes('/')) {
                let sp = line.split('/');
                if (sp.length > 1) {
                  tempTransaction.foreignCurrency = sp[1];
                  tempTransaction.foreignAmount = parseFloat(sp[0]);
                }
              } else {
                tempTransaction.amount = parseFloat(line);
                pushTransaction();
              }
            } else if (i === readingIndex + 4) {
              if (line.includes('/')) {
                let sp = line.split('/');
                if (sp.length > 1) {
                  tempTransaction.foreignCurrency = sp[1];
                  tempTransaction.foreignAmount = parseFloat(sp[0]);
                }
              } else {
                tempTransaction.amount = parseInt(line);
                pushTransaction();
              }
            } else if (i === readingIndex + 6) {
              tempTransaction.amount = parseFloat(line);
              pushTransaction();
            }
            break;
        }
      }
    }
  }
};

const extractText = async () => {
  if (!fileInput.value?.files?.[0]) {
    console.log('No file selected.');
    return;
  }

  const file = fileInput.value.files[0];
  const reader = new FileReader();
  
  reader.onload = async (event) => {
    if (!event.target?.result) {
      console.log('Failed to read the file.');
      return;
    }
    
    const pdfData = event.target.result as ArrayBuffer;
    try {
      const loadingTask = pdfjsLib.getDocument({ data: pdfData });
      const pdf = await loadingTask.promise;
      
      // let fullText = '';
  
      const numPages = pdf.numPages;
      for (let i = 1; i <= numPages; i++) {
        const page = await pdf.getPage(i);
        const textContent = await page.getTextContent();
        // fullText += textContent.items.map(item => item['str']).join('<br>');
        rawLines.value.push(...textContent.items.map(item => item['str']));
      }

      // extractedText.value = fullText;
      parseTransactions();
      console.log(transactions.value);
      console.log(cardGroups.value);
    } catch (error) {
      console.error('Error extracting text from PDF:', error);
    }
  };

  reader.readAsArrayBuffer(file);
};

</script>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
