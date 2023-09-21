

<template>
  <div class="flex w-full flex-col items-center min-h-[calc(100dvh)]" :class="[dark?'dark':'']">
    <Transition name="scalefade">
      <div tabindex="-1" aria-hidden="true" class="fixed top-0 left-0 right-0 z-50 w-full p-4 overflow-x-hidden overflow-y-auto md:inset-0 h-[calc(100%-1rem)] max-h-full flex justify-center items-center"
      v-if="showPwModal">
        <div class="relative w-full max-w-md max-h-full">
          <div class="relative bg-white rounded-lg shadow dark:bg-gray-700">
            <button type="button" class="absolute top-3 right-2.5 text-gray-400 bg-transparent hover:bg-gray-200 hover:text-gray-900 rounded-lg text-sm w-8 h-8 ml-auto inline-flex justify-center items-center dark:hover:bg-gray-600 dark:hover:text-white"
            @click="showPwModal = false">
              <svg class="w-3 h-3" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 14 14">
                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6"/>
              </svg>
              <span class="sr-only">Close modal</span>
            </button>
            <div class="px-6 py-6 lg:px-8">
              <h3 class="mb-4 text-xl font-medium text-gray-900 dark:text-white">PDF為密碼保護</h3>
              <form class="space-y-6" action="#">
                <div>
                  <label for="password" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Password</label>
                  <input type="password" name="password" id="password" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-600 dark:border-gray-500 dark:placeholder-gray-400 dark:text-white" 
                  placeholder="pdf password" ref="pw" v-model="pdfPw">
                </div>
                <button type="button" class="w-full text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
                @click="tryExtractWithPw">Submit</button>
              </form>
            </div>
          </div>
        </div>
      </div> 
    </Transition>
    <Transition name="scalefade">
      <div tabindex="-1" aria-hidden="true" class="fixed top-0 left-0 right-0 z-50 w-full p-4 overflow-x-hidden overflow-y-auto md:inset-0 h-[calc(100%-1rem)] max-h-full flex justify-center items-center"
        v-if="showNothingWarning">
        <div class="relative w-full max-w-md max-h-full">
          <div class="relative bg-white rounded-lg shadow dark:bg-gray-700">
            <button type="button" class="absolute top-3 right-2.5 text-gray-400 bg-transparent hover:bg-gray-200 hover:text-gray-900 rounded-lg text-sm w-8 h-8 ml-auto inline-flex justify-center items-center dark:hover:bg-gray-600 dark:hover:text-white"
            @click="dismissParseWarning">
              <svg class="w-3 h-3" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 14 14">
                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6"/>
              </svg>
              <span class="sr-only">Close modal</span>
            </button>
            <div class="px-6 py-6 lg:px-8">
              <h3 class="mb-4 text-xl font-medium text-gray-900 dark:text-white">無法處理檔案</h3>
              <form class="space-y-6" action="#">
                <div class="text-white">
                  請確認是否為星展銀行的信用卡帳單
                </div>
                <button type="button" class="w-full text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
                @click="dismissParseWarning">OK</button>
              </form>
            </div>
          </div>
        </div>
      </div> 
    </Transition>
    <div class="flex w-full flex-col items-center min-h-[calc(100dvh)] bg-white dark:bg-gray-900 transition-all duration-200 overflow-x-auto relative">
      <div class="flex items-center mt-4 w-9/12 lg:w-auto self-start lg:self-center ml-2 lg:ml-0">
        <input class="shrink block text-sm text-gray-900 border border-gray-300 rounded-lg cursor-pointer bg-gray-50 dark:text-gray-400 focus:outline-none dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400" id="file_input" type="file"
         @change="handleFileUpload" />
        <div class="shrink-0">
          <button type="button" 
            class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 ml-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800"
            @click="toExtractText">
            讀取
          </button>
        </div>
        <button @click="toggleDarkMode" class="border border-gray-200 dark:border-gray-700 dark:text-slate-100 text-gray-600 bg-white dark:bg-slate-800 rounded-full absolute right-2 top-2 p-1">
          <Transition name="fade" mode="out-in">
            <svg v-if="dark"
              xmlns="http://www.w3.org/2000/svg"
              class="h-6 w-6"
              viewBox="0 0 20 20"
              fill="currentColor"
            >
              <path
                fill-rule="evenodd"
                d="M10 2a1 1 0 011 1v1a1 1 0 11-2 0V3a1 1 0 011-1zm4 8a4 4 0 11-8 0 4 4 0 018 0zm-.464 4.95l.707.707a1 1 0 001.414-1.414l-.707-.707a1 1 0 00-1.414 1.414zm2.12-10.607a1 1 0 010 1.414l-.706.707a1 1 0 11-1.414-1.414l.707-.707a1 1 0 011.414 0zM17 11a1 1 0 100-2h-1a1 1 0 100 2h1zm-7 4a1 1 0 011 1v1a1 1 0 11-2 0v-1a1 1 0 011-1zM5.05 6.464A1 1 0 106.465 5.05l-.708-.707a1 1 0 00-1.414 1.414l.707.707zm1.414 8.486l-.707.707a1 1 0 01-1.414-1.414l.707-.707a1 1 0 011.414 1.414zM4 11a1 1 0 100-2H3a1 1 0 000 2h1z"
                clip-rule="evenodd"
              />
            </svg>
            <svg  v-else
              xmlns="http://www.w3.org/2000/svg"
              class="h-6 w-6"
              viewBox="0 0 20 20"
              fill="currentColor"
            >
              <path
                d="M17.293 13.293A8 8 0 016.707 2.707a8.001 8.001 0 1010.586 10.586z"
              />
            </svg>
          </Transition>
        </button>
      </div>
      <div class="w-full flex flex-col lg:flex-row mt-6 items-start lg:items-start px-4 pb-6">
        <Transition name="scalefade">
          <div class="w-full lg:w-auto p-4 bg-white border border-gray-200 rounded-lg shadow lg:p-5 dark:bg-gray-800 dark:border-gray-700" v-if="Object.keys(cardGroups).length">
            <div class="flex items-center justify-between mb-4">
                <h5 class="text-xl font-bold leading-none text-gray-900 dark:text-white">Summary</h5>
            </div>
            <div class="flow-root">
              <ul role="list" class="divide-y divide-gray-200 dark:divide-gray-700">
                <li class="py-2 sm:py-3" v-for="cg in cardGroups">
                  <div class="flex items-center space-x-6">
                    <div class="flex-1 min-w-0">
                      <p class="text-sm font-medium text-gray-900 truncate dark:text-white">
                          {{ cg.groupName}}
                      </p>
                    </div>
                    <div class="ml-2 inline-flex items-center text-base font-semibold text-gray-900 dark:text-white">
                      ${{ cg.totalAmount.toLocaleString() }}
                    </div>
                  </div>
                </li>
              </ul>
            </div>
          </div>
        </Transition>
        <Transition name="scalefade">
          <div class="mt-6 lg:mt-auto lg:ml-4 flex flex-col space-y-4 max-w-full" v-if="Object.keys(cardGroups).length">
            <div class="relative shadow-md rounded-lg border border-gray-200 dark:border-gray-700 overflow-x-auto" v-for="cg in cardGroups">
              <table class="w-full border-collapse border-spacing-1 min-w-min text-sm text-left text-gray-500 dark:text-gray-400">
                <caption class="p-4 lg:p-6 text-lg font-semibold text-left text-gray-900 bg-white dark:text-white dark:bg-gray-800">
                  {{ cg.groupName }}
                  <p class="mt-1 text-sm font-normal text-gray-500 dark:text-gray-400">${{ cg.totalAmount.toLocaleString() }}</p>
                </caption>
                <thead class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
                  <th scope="col" class="px-4 py-3 whitespace-nowrap" v-for="h in [
                    '消費明細', '消費日', '入帳日', '外幣幣別', '外幣金額', '消費金額'
                  ]"
                  >{{ h }}</th>
                </thead>
                <tbody>
                  <tr v-for="t in cg.transactions" class="bg-white border-b dark:bg-gray-800 dark:border-gray-700 hover:bg-gray-50 dark:hover:bg-gray-600 text-xs lg:text-sm last:border-b-0">
                    <th scope="row" class="px-4 lg:px-6 py-3 lg:py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white max-w-full overflow-hidden text-ellipsis">{{ t.description }}</th>
                    <td class="px-4 lg:px-6 py-3 lg:py-4">{{ t.transactionDate }}</td>
                    <td class="px-4 lg:px-6 py-3 lg:py-4">{{ t.settlementDate }}</td>
                    <td class="px-4 lg:px-6 py-3 lg:py-4">{{ t.foreignCurrency }}</td>
                    <td class="px-4 lg:px-6 py-3 lg:py-4 text-center">{{ t.foreignAmount ? `$${t.foreignAmount.toLocaleString()}` : '' }}</td>
                    <td class="px-4 lg:px-6 py-3 lg:py-4 text-center">${{ t.amount.toLocaleString() }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </Transition>
      </div>

    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue';
import * as pdfjsLib from 'pdfjs-dist';

// Initialize PDF.js
pdfjsLib.GlobalWorkerOptions.workerSrc = new URL('/assets/js/pdf.worker.min.js', import.meta.url).toString();

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

const parsed = ref<boolean>(false);

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
  parsed.value = true;
};

const needPw = ref<boolean>(false);

const showNothingWarning = ref<boolean>(false);

const extractText = async (password?: string): Promise<boolean> => {
  parsed.value = false;
  rawLines.value = [];
  transactions.value = [];
  cardGroups.value = {};

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
      const loadingTask = pdfjsLib.getDocument({ 
        data: pdfData,
        password: password,
      });
      const pdf = await loadingTask.promise;
  
      const numPages = pdf.numPages;
      for (let i = 1; i <= numPages; i++) {
        const page = await pdf.getPage(i);
        const textContent = await page.getTextContent();
        rawLines.value.push(...textContent.items.map(item => item['str']));
      }

      parseTransactions();
      console.log(transactions.value);
      console.log(cardGroups.value);
      needPw.value = false;
    } catch (error) {
      console.error(error);
      needPw.value = true;
    }
  };

  reader.readAsArrayBuffer(file);

  const w = watch(() => parsed.value, (parsed) => {
    if (parsed) {
      if (transactions.value.length === 0) {
        showNothingWarning.value = true;
      }
      w();
    }
  });
};


const dismissParseWarning = () => {
  showNothingWarning.value = false;
  parsed.value = false;
};

const pw = ref(null);
const pdfPw = ref<string | null>(null);
const showPwModal = ref<boolean>(false);

const dark = ref<boolean>(true);

const toggleDarkMode = () => {
  dark.value = !dark.value;
};

const toExtractText = async () => {
  await extractText();
  if (needPw.value) {
    showPwModal.value = true;
  } 
};

const tryExtractWithPw = () => {
  if (pdfPw.value) {
    extractText(pdfPw.value);
    showPwModal.value = false;
  }
};

</script>
