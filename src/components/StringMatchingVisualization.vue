<template>
  <div class="container">
    <header class="app-heading">Interactive and Playful Visualization of String-Matching Algorithms</header>
    <div v-if="!modeSelection">
      <button @click="modeSelection='visualize'" class="button">Single-pattern algorithms</button>
      <button @click="modeSelection='ahoCorasick'" class="button">Multi-pattern algorithms</button>
    </div>



    <div v-else>
      <button @click="modeSelection=''" class="button">Back to Menu</button>

      <!-- Aho-Corasick -->
      <div v-if="modeSelection==='ahoCorasick'">
        <svg width="100%" height="30">
          <text x="50%" y="15" fill="white" text-anchor="middle" dominant-baseline="middle" font-size="25" font-weight="bold">Aho-Corasick automaton generator</text>
        </svg>
        <div class="input-container">
          <label>Enter Dictionary (comma-separated):</label>
          <input v-model="dictionary" placeholder="e.g. tdog,dog,hot" class="input"/>
        </div>
        <div class="input-container">
          <label>Enter Input String:</label>
          <input v-model="inputString" placeholder="e.g., hotdog" class="input"/>
        </div>
        <button @click="resetGame" class="button">Reset Game</button>
        <button @click="showCorrectPath" class="button">Show correct path</button>
        <div class="message">{{ message }}</div>

        <div class="flex-container">
          <svg ref="svgRef" :id="svgRef" :width="svgWidth" :height="svgHeight" class="ahoCorasickSvg"></svg>
          <div class="graph-description">
            <ul>
              <li><strong class="grey">Grey nodes:</strong> Intermediate points on the way to a dictionary word.</li>
              <li><strong class="purple">Purple nodes:</strong> Endpoints of words from the dictionary.</li>
              <li><strong class="black">Black arrows:</strong> Show child connections, forming the trie.</li>
              <li><strong class="blue">Blue dashed arrows:</strong> Indicate suffix links for fast mismatch recovery.</li>
              <li><strong class="green">Green dashed arrows:</strong> Connect to the next node that forms a complete dictionary word via suffix links.</li>
            </ul>
          </div>
        </div>



        <div class="mascot-container">
          <div class="mascot-and-name">
            <img src="../assets/frogMascot2.webp" alt="Mascot" class="mascot"/>
            <svg width="100%" height="30">
              <text x="50%" y="15" fill="white" text-anchor="middle" dominant-baseline="middle" font-size="15" font-weight="bold">Frogbert Algorithmicus</text>
            </svg>
          </div>
          <div class="speech-bubble">To generate the Aho-Corasick automaton simply enter a dictionary above, where each word is separated by a comma (,)!
            <div class="bubble-tail"></div>
          </div>
        </div>
        <table class="comparison-results">
          <tr>
            <th>Algorithm</th>
            <th>Extension of</th>
            <th>Preprocessing Time</th>
            <th>Matching Time</th>
            <th>Space</th>
          </tr>
          <tr>
            <td>Aho–Corasick</td>
            <td>Knuth–Morris–Pratt</td>
            <td>Θ(m)</td>
            <td>Θ(n + o)</td>
            <td>Θ(m)</td>
          </tr>
          <tr>
            <td>Add more algorithms...</td>
            <td>...</td>
            <td>...</td>
            <td>...</td>
            <td>...</td>
          </tr>
        </table>
      </div>

      <div v-if="modeSelection==='visualize'">
        <label>Choose Algorithm:</label>
        <select v-model="selectedAlgorithm">
          <option value="naive">Naive Search</option>
          <option value="kmp">Knuth-Morris-Pratt</option>
          <option value="rabinkarp">RabinKarp</option>
        </select>

        <input v-model="haystack" placeholder="Enter main string" class="input"/>
        <input v-model="needle" placeholder="Enter pattern string" class="input"/>
        <div>
          <label for="speed">Pause between steps: {{speed}}ms</label>
          <input type="range" id="speed" v-model="speed" min="100" max="2000" class="slider" />
        </div>
        <button class="button" @click="startSearch">Start</button>
        <button class="button" @click="pauseSimulation">Pause</button>
        <button class="button" @click="stepBackward">Step Backward</button>
        <button class="button" @click="stepForward">Step Forward</button>

        <!-- User LPS-table -->
        <div v-if="selectedAlgorithm === 'kmp'" class="partial-match-table">
          <table class="lps-table">
            <tr>
              <th>i</th>
              <th v-for="(value, index) in needle" :key="`header-${index}`">
                [{{ index }}]
              </th>
            </tr>
            <tr>
              <td>needle[i]</td>
              <td v-for="(value, index) in needle" :key="`s-${index}`">
                {{ value }}
              </td>
            </tr>
            <tr>
              <td>π(i)</td>
              <td v-for="(value, index) in lpsTable" :key="`pi-${index}`">
                <input v-if="!showLPS" type="number" v-model="userLPS[index]" class="lps-input" />
                <span v-else :class="{ 'blurred-text': !showLPS }">{{ value }}</span>
              </td>
            </tr>
          </table>
          <button @click="toggleLPSVisibility" class="button">{{ showLPS ? 'Hide LPS Values' : 'Show LPS Values' }}</button>
          <button @click="checkLPS" class="button">Check</button>
        </div>

        <div class="label"></div>
        <svg :width="haystack.length * 60 + 100" height="250" class="svg">
          <!-- Render haystack -->
          <template v-for="(char, index) in haystack" :key="`haystack-${index}`">
            <rect :x="index * 50" y="20" width="50" height="50" :fill="getHaystackColor(index)" stroke="white" />
            <text :x="index * 50 + 25" y="50" fill="white" text-anchor="middle" dominant-baseline="middle">{{ char }}</text>
            <!-- Index label -->
            <text :x="index * 50 + 25" y="80" fill="white" text-anchor="middle" dominant-baseline="middle">[{{ index }}]</text>
          </template>
          <text :x="36" y="100" fill="white" text-anchor="middle" dominant-baseline="middle" font-size="15" font-weight="bold">Haystack</text>

          <g :style="{ transition: 'transform ' + speed / 1000 + 's ease', transform: 'translateX(' + patternOffset * 50 + 'px)' }">
            <!-- Render needle -->
            <template v-for="(char, index) in needle" :key="`needle-${index}`">
              <rect :x="index * 50" y="140" width="50" height="50" :fill="getPatternColor(index)" stroke="white" />
              <text :x="index * 50 + 25" y="170" fill="white" text-anchor="middle" dominant-baseline="middle">{{ char }}</text>
              <!-- Index label -->
              <text :x="index * 50 + 25" y="200" fill="white" text-anchor="middle" dominant-baseline="middle">[{{ index }}]</text>
            </template>
            <text :x="30" y="220" fill="white" text-anchor="middle" dominant-baseline="middle" font-size="15" font-weight="bold">Needle</text>
            <template v-if="selectedAlgorithm === 'rabinkarp'">
              <!-- needle hash value -->
              <rect x="0" y="115" width="150" height="30" fill="white" stroke="black" rx="15" ry="15" />
              <text x="10" y="125" fill="black" text-anchor="start" dominant-baseline="hanging">
                Hash:{{ needleHash }}
              </text>
            </template>
          </g>
          <template v-if="selectedAlgorithm === 'rabinkarp'">
            <!-- haystack hash value with conditional coloring -->
            <rect :x="patternOffset * 50" y="-5" width="150" height="30" fill="white" stroke="black" rx="15" ry="15" />
            <text :x="patternOffset * 50 + 10" y="15" fill="black" text-anchor="start" dominant-baseline="baseline">
              Hash:
              <template v-if="haystackHash === needleHash">
                <!-- Entire hash in green if they match -->
                <tspan fill="green">{{ haystackHash }}</tspan>
              </template>
              <template v-else>
                <!-- Colored based on differences if they don't match -->
                <tspan v-for="(digit, index) in getColoredHash(haystackHash.toString(), needleHash.toString())" :fill="digit.color" :key="index">
                  {{ digit.char }}
                </tspan>
              </template>
            </text>
            <!-- Conditionally rendered icon inside the haystack bubble -->
            <g :transform="`translate(${patternOffset * 50 + 130}, 0)`">
              <template v-if="haystackHash === needleHash">
                <!-- Green checkmark, shifted 5px down -->
                <g class="tooltip-container">
                  <circle cx="0" cy="10" r="10" fill="green" />
                  <path d="M-5 10 l5 5 l10 -10" stroke="white" stroke-width="2"/>
                  <title>Now the hash values DO match</title>
                </g>
              </template>
              <template v-else>
                <!-- Red cross, positioned correctly -->
                <g class="tooltip-container">
                  <circle cx="0" cy="10" r="10" fill="red" />
                  <path d="M-5 5 l10 10" stroke="white" stroke-width="2"/>
                  <path d="M5 5 l-10 10" stroke="white" stroke-width="2"/>
                  <title>The hash values do not match</title>
                </g>
              </template>
            </g>
          </template>

        </svg>
        <div class="mascot-container">
          <div class="mascot-and-name">
            <img src="../assets/frogMascot2.webp" alt="Mascot" class="mascot"/>
            <svg width="100%" height="30">
              <text x="50%" y="15" fill="white" text-anchor="middle" dominant-baseline="middle" font-size="15" font-weight="bold">Frogbert Algorithmicus</text>
            </svg>
          </div>
          <div class="speech-bubble">{{ searchComment }}
            <div class="bubble-tail"></div>
          </div>
        </div>
        <div class="comparison-table">
          <table class="comparison-results">
            <tr>
              <th>Algorithm</th>
              <th>Preprocessing Time</th>
              <th>Matching Time</th>
              <th>Comparisons until termination</th>
            </tr>
            <tr>
              <td>Naive Search</td>
              <td>none</td>
              <td>O(n+m) average, O(mn) worst</td>
              <td>{{ comparisonResults.naive }}</td>
            </tr>
            <tr>
              <td>Knuth-Morris-Pratt (KMP)</td>
              <td>Θ(m) (LPS-table)</td>
              <td>Θ(n+m)</td>
              <td>{{ comparisonResults.kmp }}</td>
            </tr>
            <tr>
              <td>Rabin-Karp</td>
              <td>Θ(m) (NeedleHash)</td>
              <td>Θ(n) average, Θ(mn) worst</td>
              <td>{{ comparisonResults.rabinkarp }}</td>
            </tr>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { graphviz } from 'd3-graphviz';
import * as d3 from "d3";

const modeSelection = ref('') //holds selected mode (visualization, quiz, etc.)
const selectedAlgorithm = ref('naive') //holds selected algorithm
const haystack = ref('abcabcabcd')
const needle = ref('abcd')
const speed = ref(500) // stores pause between steps
const steps = ref([]) // array that holds the sequence of steps/states for visualization
const currentStepIndex = ref(-1) // keeps track of current step -1 is before start
const simulationInterval = ref(null) // setInterval(function, delay); executes function every "delay" ms.
const patternOffset = ref(0) // indicates current position of the needle relative to haystack
const currentMatch = ref([]) // stores for every step which characters match
const searchComment = ref('Click "Start" to execute the chosen algorithm.') // holds commentary related to the current step of the algorithm
const algorithmExplanation = ref('') // contains explanation to the selected algorithm
const comparisonIndices = ref([]) // contains indices of characters that are compared (for highlighting)
const haystackHash = ref('')
const needleHash = ref('')
const lpsTable = ref([]) // correct LPS table
const userLPS = ref([]) // user answers for LPS table
const showLPS = ref(true)  // visibility switch for showing/hiding LPS values
const comparisonCount = ref(0) // contains number of comparisons
const comparisonResults = ref({
  naive: 0,
  kmp: 0,
  rabinkarp: 0
});
const dictionary = ref('')
const ahoCorasick = ref(null)
const svgWidth = ref(800)
const svgHeight = ref(600)
const svgRef = ref(null);
const inputString = ref('')
const currentPathIndex = ref(0)
const gameOver = ref(false)
const message = ref('')


// initialize userLPS with zeros based on needle length
userLPS.value = Array(needle.value.length).fill(0);

// ---------------------------------------- SINGLE PATTERN ALGORITHMS ----------------------------------------

// NAIVE
function precomputeNaiveSteps() {
  steps.value = [];
  let m = haystack.value.length;
  let p = needle.value.length;
  comparisonCount.value = 0;

  if (m < p) {
    // Exit if needle is longer than haystack
    steps.value.push({ patternOffset: 0, currentMatch: [], searchComment: 'String not found!', found: false });
    return;
  }

  for (let i = 0; i <= m - p; i++) {
    let allMatched = true;
    // Initialize comparisonIndices for the current window of comparison
    comparisonIndices.value = Array.from({length: p}, (_, k) => i + k);

    for (let j = 0; j < p; j++) {
      // Copy matches from last step or create a fresh array if it's the first comparison
      let currentMatchState = j > 0 ? steps.value[steps.value.length - 1].currentMatch.slice() : new Array(p).fill(null);

      comparisonCount.value++;

      if (haystack.value[i + j] === needle.value[j]) {
        currentMatchState[j] = true;
        steps.value.push({
          patternOffset: i,
          currentMatch: currentMatchState,
          found: false,
          searchComment: `Character match at haystack[${i + j}] and needle[${j}].`
        });
      } else {
        currentMatchState[j] = false;
        allMatched = false;
        steps.value.push({
          patternOffset: i,
          currentMatch: currentMatchState,
          found: false,
          searchComment: `Mismatch at haystack[${i + j}] and needle[${j}].`
        });
        break; // Exit inner loop on first mismatch
      }
    }

    // Clear comparisonIndices after each outer loop iteration
    comparisonIndices.value = [];

    // If we completed the inner loop without breaking, it's a complete match
    if (allMatched) {
      steps.value[steps.value.length - 1].found = true; // Mark last step as found
      steps.value[steps.value.length - 1].searchComment = `Needle found at haystack index ${i} after ${comparisonCount.value} comparisons. (Naive)`;
      break; // Stop search
    }
  }

  // Final step -> no match found
  if (!steps.value.some(step => step.found)) {
    steps.value.push({ patternOffset: 0, currentMatch: [], searchComment: 'String not found!', found: false });
  }
}

// KMP
function precomputeKMPSteps() {
  const pat = needle.value;
  const txt = haystack.value;
  const M = pat.length;
  const N = txt.length;
  const lps = new Array(M).fill(0);
  comparisonCount.value = 0;

  // Compute the LPS array.
  computeLPSArray(pat, M, lps);
  lpsTable.value = [...lps]; // Assign the computed LPS values for reactivity

  steps.value = [];
  let i = 0; // index for haystack
  let j = 0; // index for needle

  if (M > N) {
    steps.value.push({ patternOffset: 0, currentMatch: [], searchComment: 'String not found!', found: false });
    return;
  }

  while (i < N) {
    comparisonCount.value++;
    if (pat[j] === txt[i]) {
      steps.value.push({
        patternOffset: i - j,
        currentMatch: Array.from({length: M}, (_, k) => k <= j ? true : null),
        found: false,
        searchComment: `Character match at haystack[${i}] and needle[${j}].`
      });
      i++;
      j++;
      if (j === M) {
        steps.value.push({
          patternOffset: i - j,
          currentMatch: new Array(M).fill(true),
          found: true,
          searchComment: `Needle found at haystack index ${i - j} after ${comparisonCount.value} comparisons. (KMP)`
        });
        // After finding a match, we consult the LPS table for where to continue the search
        j = lps[j - 1];
        break;
      }
    } else {
      if (j !== 0) {
        // Mismatch after at least one matching character
        steps.value.push({
          patternOffset: i - j,
          currentMatch: Array.from({ length: M }, (_, k) => k < j ? true : null),
          found: false,
          searchComment: `Mismatch at haystack[${i}] and needle[${j}]. Consulting LPS table with index of last correct match: π(${j}-1) -> π(${j-1}): ${lps[j-1]}, so we shift needle with index ${lps[j-1]} to index ${i} of haystack.`
        });
        // We will consult the LPS table and not increase i in this case
        j = lps[j - 1];
      } else {
        // Mismatch at the beginning of the pattern, move to the next character in the text
        steps.value.push({
          patternOffset: i,
          currentMatch: new Array(M).fill(null),
          found: false,
          searchComment: `Mismatch at haystack[${i}] and needle[0]. Shift needle by one to index 0.`
        });
        i++;
      }
    }
    // Check if the remaining substring is shorter than the pattern length
    if (N - i < M - j) {
      steps.value.push({ patternOffset: i, currentMatch: new Array(M).fill(null), searchComment: 'String not found!', found: false });
      break;
    }
  }

  if (!steps.value.some(step => step.found)) {
    steps.value.push({ patternOffset: 0, currentMatch: [], searchComment: 'No match found.', found: false });
  }
}

//RABIN KARP
function precomputeRabinKarpSteps() {
  const m = haystack.value.length;
  const p = needle.value.length;
  needleHash.value = hash(needle.value);  // Set initial hash for the needle
  let previousHash = null;
  comparisonCount.value = 0;

  steps.value = [];  // Reset steps

  if (m < p) {
    steps.value.push({ patternOffset: 0, currentMatch: [], searchComment: 'Pattern is longer than text. No search possible.', found: false });
    return;
  }

  for (let i = 0; i <= m - p; i++) {
    const haystackSubstring = haystack.value.substring(i, i + p);
    const currentHash = previousHash === null ? hash(haystackSubstring) : rehash(previousHash, haystack.value[i - 1], haystack.value[i + p - 1], p);
    previousHash = currentHash;  // Update previousHash to current for the next iteration
    comparisonCount.value++;

    // Push the step showing hash comparison at this index
    steps.value.push({
      patternOffset: i,
      currentMatch: new Array(p).fill(null),
      found: false,
      searchComment: `Hash comparison at index ${i}`,
      currentHaystackHash: currentHash
    });

    if (currentHash === needleHash.value) {
      // Before character comparison, note that hashes are now equal
      steps.value.push({
        patternOffset: i,
        currentMatch: new Array(p).fill(null),
        found: false,
        searchComment: "Hashes are now equal. Now we compare the individual characters.",
        currentHaystackHash: currentHash
      });

      let currentMatchState = new Array(p).fill(null);
      let allCharactersMatch = true;
      for (let j = 0; j < p; j++) {
        const match = haystack.value[i + j] === needle.value[j];
        currentMatchState[j] = match;  // Update the match state for the current character
        allCharactersMatch &= match;  // Determine if all characters match
        comparisonCount.value++;
        steps.value.push({
          patternOffset: i,
          currentMatch: currentMatchState.slice(),  // Use slice to create a copy of the array
          found: false,
          searchComment: `Comparing character ${j}: '${haystack.value[i + j]}' with '${needle.value[j]}'`,
          currentHaystackHash: currentHash
        });
      }

      if (allCharactersMatch) {
        steps.value.push({
          patternOffset: i,
          currentMatch: new Array(p).fill(true),
          found: true,
          searchComment: `Complete match found after ${comparisonCount.value} comparisons. (RabinKarp)`,
          currentHaystackHash: currentHash
        });
        break; // Stop further processing as we found a match
      }
    }
  }

  // Final step if no match is found throughout
  if (!steps.value.some(step => step.found)) {
    steps.value.push({ patternOffset: 0, currentMatch: [], searchComment: 'No match found in the entire text.', found: false, currentHaystackHash: 0 });
  }
}


// ---------------------------------------- VISUALIZATION  ----------------------------------------

// picks the chosen algorithm for execution
function startSearch() {
  if (!haystack.value.trim() || !needle.value.trim()) {
    searchComment.value = "Please enter both a main string and a pattern string.";
    return;
  }

  // reset simulation
  currentStepIndex.value = 0;

  if (selectedAlgorithm.value === 'naive') {
    precomputeNaiveSteps();
  } else if (selectedAlgorithm.value === 'rabinkarp') {
    precomputeRabinKarpSteps();
  } else if (selectedAlgorithm.value === 'kmp') {
    precomputeKMPSteps();
  }

  runSimulation(); // starts simulation based steps we just computed
}

// handles automatic playback -> setInterval(function, delay);
function runSimulation() {
  pauseSimulation(); // clear any existing intervals
  simulationInterval.value = setInterval(() => {
    if (currentStepIndex.value < steps.value.length - 1) {
      // steps forward every {{speed.value}} milliseconds until end
      currentStepIndex.value++;
    } else {
      pauseSimulation();
    }
  }, speed.value);
}

// pause automatic playback
function pauseSimulation() {
  clearInterval(simulationInterval.value);
}

// increases the index of our current position in the steps[] array
function stepForward() {
  pauseSimulation(); // ensure we stop automatic playback when stepping manually
  if (currentStepIndex.value < steps.value.length - 1) {
    currentStepIndex.value++;
  }
}

// decreases the index of our current position in the steps[] array
function stepBackward() {
  pauseSimulation();
  if (currentStepIndex.value > 0) {
    currentStepIndex.value--;
  }
}

// reads data from step array and modifies reactive fields so that view is updated
function visualizeStep() {
  const step = steps.value[currentStepIndex.value];
  if (step) {
    patternOffset.value = step.patternOffset;
    currentMatch.value = step.currentMatch;
    searchComment.value = step.searchComment;
    haystackHash.value = step.currentHaystackHash;
    comparisonIndices.value = step.currentMatch.map((match, idx) => match !== null ? patternOffset.value + idx : null).filter(x => x !== null);
  } else {
    comparisonIndices.value = []; // Reset on steps without active comparisons
  }
}

// ---------------------------------------- WATCHERS ----------------------------------------

// executes visualizeStep() every time "currentStepIndex" is changed
watch(currentStepIndex, () => {
  visualizeStep();
}, { immediate: true }); // use "immediate: true" to start visualization before first user interaction

watch (selectedAlgorithm, () => {
  startSearch();
});

// so table gets updated instantly when needle string changes
watch (needle, () => {
  precomputeKMPSteps();
});

watch(selectedAlgorithm, (newAlgorithm) => {
  if (newAlgorithm === 'naive') {
    algorithmExplanation.value = "The Naive Search algorithm...";
  } else if (newAlgorithm === 'kmp') {
    algorithmExplanation.value = "The Knuth-Morris-Pratt algorithm...";
  } else if (newAlgorithm === 'rabinkarp') {
    algorithmExplanation.value = "The Rabin-Karp algorithm uses hash functions...";
  } else {
    algorithmExplanation.value = ''; // default text
  }
}, { immediate: true }); // run immediately


// update steps table when needle or haystack changes
watch([haystack, needle], () => {
  if (haystack.value && needle.value) {
    computeAllComparisons();
  }
}, { immediate: true }); // run immediately

//updates AHOCORASICK graph when dictionary changes
watch(dictionary, newValue => {
  if (newValue) {
    // Split the input string into words, trim whitespace, and sort alphabetically
    const words = newValue.split(',').map(s => s.trim()).sort();

    ahoCorasick.value = new AhoCorasickAutomaton();
    ahoCorasick.value.buildTrie(words);
    const dotString = ahoCorasick.value.generateDot();
    renderGraph(dotString);

    // Initialize the correct path for the visualization after the graph is built
    if (inputString.value) {
      ahoCorasick.value.initializeCorrectPath();
    }
  }
}, { immediate: true });

//updates AHOCORASICK graph when inputString changes
watch([dictionary, inputString], () => {
  if (dictionary.value && inputString.value) {
    // Ensure the words are trimmed and sorted alphabetically every time
    const words = dictionary.value.split(',').map(s => s.trim()).sort();

    ahoCorasick.value = new AhoCorasickAutomaton();
    ahoCorasick.value.buildTrie(words);
    ahoCorasick.value.initializeCorrectPath();
    const dotString = ahoCorasick.value.generateDot();
    renderGraph(dotString);

    currentPathIndex.value = 0;
    gameOver.value = false;
    message.value = "Start clicking the nodes in the correct order!";
  }
}, { immediate: true });

// ---------------------------------------- OTHER FUNCTIONS ----------------------------------------

// method to calculate and store the LPS-table used in the KMP algorithm into an array
function computeLPSArray(pat, M, lps) {
  let len = 0; // length of the previous longest prefix suffix
  let i = 1;
  lps[0] = 0; // lps[0] is always 0

  while (i < M) {
    if (pat[i] === pat[len]) {
      len++;
      lps[i] = len;
      lpsTable.value[i] = len;
      i++;
    } else {
      if (len !== 0) {
        len = lps[len - 1];
      } else {
        lps[i] = 0;
        i++;
      }
    }
  }
}

// method to check user inputs against the actual LPS values stored in lpsTable
function checkLPS() {
  let allCorrect = true;
  for (let i = 0; i < needle.value.length; i++) {
    // Ensure that userLPS values are integers and compare them to lpsTable
    if (parseInt(userLPS.value[i], 10) !== lpsTable.value[i]) {
      alert(`Incorrect value at index ${i}. Expected ${lpsTable.value[i]}, but got ${userLPS.value[i]}`);
      allCorrect = false;
      break; // Stop checking after the first incorrect value
    }
  }
  if (allCorrect) {
    alert('All values are correct!');
  }
}

// toggle logic for button to show LPS solution
function toggleLPSVisibility() {
  showLPS.value = !showLPS.value;
}

// maps true -> green & false -> red
const getPatternColor = (index) => {
  if (currentMatch.value[index] === true) return 'green';
  if (currentMatch.value[index] === false) return 'red';
  return 'grey';
};

// used to highlight characters in the haystack in the same color as the needle or yellow for current hash calculation
const getHaystackColor = (index) => {
  let needleIndex = index - patternOffset.value;
  if (needleIndex >= 0 && needleIndex < needle.value.length) {
    if (currentMatch.value[needleIndex] === true) {
      return 'green';
    }
    if (currentMatch.value[needleIndex] === false) {
      return 'red';
    }
    // Highlight current hash comparison characters for Rabin-Karp
    if (selectedAlgorithm.value === 'rabinkarp') {
      return 'grey';
    }
  }
  return '#1E1E1E';
};

//helper function to highlight digits that are different when comparing the hash values of haystack and needle
const getColoredHash = (haystackHash, needleHash) => {
  const maxLength = Math.max(haystackHash.length, needleHash.length);
  let coloredHash = [];

  for (let i = 0; i < maxLength; i++) {
    if (haystackHash[i] && needleHash[i] && haystackHash[i] !== needleHash[i]) {
      // color different digits red
      coloredHash.push({ char: haystackHash[i], color: 'red' });
    } else if (haystackHash[i]) {
      // color equal digits black
      coloredHash.push({ char: haystackHash[i], color: 'black' });
    }
  }
  return coloredHash;
};

function computeAllComparisons() {
  precomputeNaiveSteps();
  comparisonResults.value.naive = comparisonCount.value;
  precomputeKMPSteps();
  comparisonResults.value.kmp = comparisonCount.value;
  precomputeRabinKarpSteps();
  comparisonResults.value.rabinkarp = comparisonCount.value;
}

// ---------------------------------------- HASHING ----------------------------------------

const base = 256;
const modulus = 1000000009;

function hash(s) {
  let hashValue = 0;
  for (let i = 0; i < s.length; i++) {
    hashValue = (hashValue * base + s.charCodeAt(i)) % modulus;
  }
  return hashValue;
}

//ROLLING HASH
// updates hash when shifting needle by subtracting the dropped left character
// and adding the new right character
function rehash(oldHash, droppedChar, addedChar, patternLength) {
  let newHash = oldHash - (droppedChar.charCodeAt(0) * Math.pow(base, patternLength - 1)) % modulus;
  newHash = (newHash * base + addedChar.charCodeAt(0)) % modulus;
  // ensure positive hash value
  if (newHash < 0) newHash += modulus;
  return newHash;
}

// ---------------------------------------- AHO-CORASICK ----------------------------------------

class TrieNode {
  constructor(index) {
    this.children = {};
    this.fail = null;
    this.outputs = [];
    this.index = index;
    this.isEndOfWord = false;
  }
}

class AhoCorasickAutomaton {
  constructor() {
    this.nodeIndex = 0;
    this.root = new TrieNode(this.nodeIndex++);
  }

  insert(word) {
    let node = this.root;
    for (const char of word) {
      if (!node.children[char]) {
        node.children[char] = new TrieNode(this.nodeIndex++);
      }
      node = node.children[char];
    }
    node.isEndOfWord = true;
    node.outputs.push(word);
  }

  buildFailureLinks() {
    const queue = [this.root];
    while (queue.length > 0) {
      const current = queue.shift();
      for (const [char, child] of Object.entries(current.children)) {
        if (current === this.root) {
          child.fail = this.root;
        } else {
          let failNode = current.fail;
          while (failNode !== null && !failNode.children[char]) {
            failNode = failNode.fail;
          }
          child.fail = failNode ? failNode.children[char] : this.root;
        }
        child.outputs = child.outputs.concat(child.fail.outputs);
        queue.push(child);
      }
    }
  }

  buildTrie(words) {
    for (const word of words) {
      this.insert(word);
    }
    this.buildFailureLinks();
  }

  initializeCorrectPath() {
    this.correctPath = [];
    let node = this.root;
    this.correctPath.push(node.index); // Start at the root node

    for (let char of inputString.value) {
      // Proceed to next node if possible
      while (node !== this.root && !node.children[char]) {
        node = node.fail;  // Follow failure link
        this.correctPath.push(node.index);
      }
      if (node.children[char]) {
        node = node.children[char];  // Move to the child node
        this.correctPath.push(node.index);
      }
    }

    // After processing the string, traverse up using fail links to find all possible suffixes
    let lastNode = node;
    while (node !== this.root) {
      node = node.fail;
      if (node.isEndOfWord) {
        this.correctPath.push(node.index);
        console.log("Suffix link to end word node:", node.index);
        if (node === lastNode) {  // Avoid loops where a node points to itself
          break;
        }
        lastNode = node;
      }
    }

    console.log("Final correct path:", this.correctPath);
  }

  generateDot() {
    let dot = 'digraph Trie {\n';
    //dot += 'graph [splines=polyline];\n';  // edges straight but segmented
    dot += 'node [shape=circle, style=filled, fillcolor=lightgrey];\n';
    dot += 'edge [fontcolor=black];\n';

    const queue = [{node: this.root, path: ''}];
    let index = 0;
    this.root.index = index;

    while (queue.length) {
      const {node, path} = queue.shift();

      // only color node if it's end of a dictionary word
      if (node.isEndOfWord) {
        dot += `${node.index} [label="${path}", fillcolor="#9d9dc7"];\n`;
      } else {
        dot += `${node.index} [label="${path}"];\n`;
      }

      for (const [char, child] of Object.entries(node.children)) {
        if (child.index === undefined) {
          child.index = ++index;
        }
        queue.push({node: child, path: path + char});
        dot += `${node.index} -> ${child.index} [label="${char}", color=black];\n`;
      }

      if (node.fail && node !== this.root) {
        dot += `${node.index} -> ${node.fail.index} [style=dashed, color=blue, constraint=false];\n`;
      }

      let dictSuffix = node.fail;
      while (dictSuffix && dictSuffix !== this.root && !dictSuffix.isEndOfWord) {
        dictSuffix = dictSuffix.fail;
      }
      if (dictSuffix && dictSuffix.isEndOfWord) {
        dot += `${node.index} -> ${dictSuffix.index} [style=dashed, color=green, constraint=false];\n`;
      }
    }

    dot += '}\n';
    return dot;
  }
}

function renderGraph(dotString) {
  if (svgRef.value) {
    graphviz(svgRef.value)
        .transition(() => d3.transition().duration(500))
        .renderDot(dotString)
        .on("end", () => {
          // make nodes clickable
          d3.selectAll(".node").style("cursor", "pointer").on("click", function(event) {
            const d = d3.select(this).datum();
            handleNodeClick(event, d);
          });
          // deactivate doubleclick zoom
          d3.select('svg').on('dblclick.zoom', null);
        })
        .fit(true)
        .scale(1);
  }
  adjustViewBox();
}

function showCorrectPath() {
  message.value = "Correct path: " + ahoCorasick.value.correctPath.join(" -> ");
}

function handleNodeClick(event, d) {
  // Prevent interaction if the game is over
  if (gameOver.value) return;

  // Get the index of the clicked node
  const clickedNodeIndex = parseInt(d.key);

  // Check if the clicked node is the next correct node in the path
  if (clickedNodeIndex === ahoCorasick.value.correctPath[currentPathIndex.value]) {
    // Color the node green to indicate correct selection
    d3.select(event.currentTarget).select("ellipse, circle").style("fill", "#6fdc6f");

    // Increment the current path index to move to the next step
    currentPathIndex.value++;

    // Update the message to show progress and encourage continuation
    message.value = `Correct! Continue. (${currentPathIndex.value}/${ahoCorasick.value.correctPath.length})`;

    // Check if the end of the path is reached to conclude the game
    if (currentPathIndex.value === ahoCorasick.value.correctPath.length) {
      gameOver.value = true;  // Set the game state to over
      message.value = "Congratulations! You have completed the Aho-Corasick path!";
    }
  } else {
    // Update the message to indicate the wrong node was clicked without resetting the game
    message.value = "Wrong node. Try again!";
  }
}


function resetGame() {
  currentPathIndex.value = 0;
  gameOver.value = false;
  message.value = "Game reset. Start clicking the nodes in the correct order!";
  const dotString = ahoCorasick.value.generateDot();
  renderGraph(dotString);
}


function adjustViewBox() {
  if (svgRef.value) {
    let bbox = svgRef.value.getBBox();
    svgRef.value.setAttribute('viewBox', `${bbox.x - 10} ${bbox.y - 10} ${bbox.width + 20} ${bbox.height + 20}`);
  }
}

</script>



<style>
:root {
  --theme-color: #606090;
}

.container {
  background-color: black;
  margin: auto;
  max-width: 3000px;
  color: white;
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.app-heading {
  color: #9f9fed;
  text-align: center;
  font-size: 2em;
  font-weight: bold;
  margin: 20px 0;
  text-shadow: 2px 2px 4px #000;
}

.input {
  margin: 10px;
  background-color: var(--theme-color);
  color: white;
  border: 1px solid #555;
  padding: 8px;
  border-radius: 4px;
}

.button {
  margin: 10px;
  background-color: var(--theme-color);
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.slider {
  appearance: none;
  width: 100%;
  height: 8px;
  background: var(--theme-color);
  outline: none;
  opacity: 0.7;
  -webkit-transition: .2s;
  transition: opacity .2s;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  background: #601099;
  cursor: pointer;
}

.button:hover {
  background-color: #2E8B57;
  cursor: pointer;
}

.svg {
  border: 1px solid var(--theme-color);
  margin-top: 20px;
}

.ahoCorasickSvg {
  border: 1px solid var(--theme-color);
  margin-top: 20px;
  background-color: white;
}

.search-comment {
  margin-top: 20px;
  color: #FFFFFF;
  font-size: 1rem;
  text-align: center;
}

.algorithm-explanation {
  margin-bottom: 20px;
  padding: 10px;
  background-color: var(--theme-color);
  color: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  font-size: 1rem;
  line-height: 1.4;
  text-align: center;
}

.partial-match-table {
  margin-top: 20px;
  overflow-x: auto;
}

.lps-table {
  border-collapse: collapse;
  width: 100%;
  background-color: white;
  color: black;
  margin: auto;
}

.lps-input {
  width: 30px;
  height: 20px;
  padding: 4px 6px;
  margin: auto;
  display: block;
  text-align: center;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.lps-table th,
.lps-table td {
  border: 1px solid black;
  padding: 8px;
  text-align: center;
}

.lps-table th {
  background-color: var(--theme-color);
  font-weight: bold;
  color: white;
}

.lps-table td:nth-child(even) {
  background-color: #f9f9f9;
}

.blurred-text {
  color: transparent;
  text-shadow: 0 0 8px rgba(0,0,0,0.5);
}

.tooltip-container {
  cursor: help;
}

.mascot-container {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 20px;
}

.mascot-and-name {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-right: 10px;
}

.mascot {
  width: 100px;
  height: auto;
  transition: transform 0.3s ease;
}

.speech-bubble {
  position: relative;
  background-color: #f0f0f0;
  border-radius: 8px;
  padding: 10px;
  color: black;
  text-align: center;
  max-width: 250px;
  box-shadow: 2px 2px 2px rgba(0, 0, 0, 0.2);
  min-width: 150px;
  min-height: 50px;
}

.speech-bubble::before {
  content: '';
  position: absolute;
  left: -90px;
  top: 50%;
  width: 0;
  height: 0;
  border-style: solid;
  border-width: 15px 90px 15px 0;
  border-color: transparent #f0f0f0 transparent transparent;
  transform: translateY(-50%);
}

.comparison-table {
  margin-top: 20px;
  color: white;
}

.comparison-results {
  border-collapse: collapse;
  width: 100%;
  margin-top: 10px;
}

.comparison-results th,
.comparison-results td {
  border: 1px solid #fff;
  padding: 8px;
  text-align: left;
}

.comparison-results th {
  background-color: var(--theme-color);
}

.comparison-results td {
  background-color: #f2f2f2;
  color: black;
}

.input-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20px;
}

.ahoCorasickSvg, .graph-description {
  display: inline-block;
  vertical-align: top;
}

.flex-container {
  display: flex;
  align-items: stretch;
}

.ahoCorasickSvg {
  flex: 1;
  min-height: 600px;
  pointer-events: none;
}

.graph-description {
  max-width: 220px;
  margin-left: 20px;
  padding: 10px;
  background-color: #f9f9f9;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  color: #333;
  flex-shrink: 0;
  height: auto;
  text-align: left;
}

.graph-description ul {
  list-style-type: none;
  padding: 0;
}

.graph-description li {
  margin-bottom: 10px;
}

.graph-description strong {
  display: block;
}

.grey {
  color: grey;
}

.purple {
  color: #6363e0;
}

.black {
  color: black;
}

.blue {
  color: blue;
}

.green {
  color: green;
}

.ahoCorasickSvg {
  border: 1px solid var(--theme-color);
  margin-top: 20px;
  background-color: white;
  pointer-events: all;
}

.message {
  margin-top: 10px;
  padding: 10px;
  background-color: #f0f0f0;
  border-radius: 5px;
  color: black;
  text-align: center;
}

svg .graph .node {
  pointer-events: auto;
}

svg .graph .edge {
  pointer-events: none;
}

svg {
  -webkit-user-select: none; /* Chrome, Safari, Opera */
  -moz-user-select: none;    /* Firefox */
  -ms-user-select: none;     /* Internet Explorer/Edge */
  user-select: none;         /* Nicht-präfixierte Version, derzeit in allen modernen Browsern unterstützt */

  pointer-events: none;      /* Deaktiviert Mausereignisse wie Klicken, Scrollen und Draggen */
}


</style>