<script setup>
import { onMounted, ref, onUnmounted } from 'vue';

const limit = ref(9);

let numSelected = ref(null);
let cellSelected = ref(null);

let allCells = ref([]); //array di tutte le celle
let digitNumbers = ref([]); //array di tutti i pulsanti dei numeri

let errors = ref(0);

// per la fine del gioco
let allCellStart = ref([]); // tutte le celle iniziali
let emptyCellCount = ref(0); // tutte le celle vuoute
let correctCount = ref(0); //counter per le celle giuste inserite
let isGameOver = ref(false);

// ARRAY che contine tutti i contatori dei numeri
let counters = [];


const board = [
  "-26---81-",
  "3--7-8--6",
  "4---5---7",
  "-5-1-7-9-",
  "--39-51--",
  "-4-3-2-5-",
  "1---3---2",
  "5--2-4--9",
  "-38---46-"
];

const solution = [
  "726493815",
  "315728946",
  "489651237",
  "852147693",
  "673985124",
  "941362758",
  "194836572",
  "567214389",
  "238579461"
];

onMounted(() => {
  document.addEventListener('click', handleClickOutsideCell);
  allCells = document.querySelectorAll(".cell");
  digitNumbers = document.querySelectorAll('.number');
  allCellStart = document.querySelectorAll('.cell-start');
  emptyCellCount = (limit.value * limit.value) - allCellStart.length; //calcolo quante celle vuote ci sono

  // inizializza tutti i contatori a 0
  for (let i = 0; i <= 9; i++) {
    counters.push(ref(0)); 
  }

  // ciclo tutte le celle, aumento il counter del numero che sta all'interno della cella
  allCells.forEach(cell => {
    for (let i = 1; i <= 9; i++) {
      if (parseInt(cell.innerText) === i) {
        // eval(`counter${i}.value++`);
        // break; 
        incrementCounter(i);
      }
    }
  });

  // console.table(solution);

});

window.onload = function(){
  // console.log("funzione generata dopo il mounted");
}

function incrementCounter(number) {
  counters[number].value++;
}

// funzione richiamata goni volta che si clicca
function handleClickOutsideCell(event) {
  const isClickedInsideCell = event.target.classList.contains('cell');
  const isClickedInsideButton = event.target.classList.contains('number');
  if (!isClickedInsideCell && !isClickedInsideButton) {
    resetCellStyles();
  }
}

function resetCellStyles() {

  // Rimuovi gli stili dalle celle
  allCells.forEach(cell => {
    cell.style.cssText = "";
    cell.classList.remove('equal-number');
    cell.classList.remove('cell-selected');
    cell.classList.remove('row-col-selected'); 
    cell.classList.remove('cell-start-selected'); 

    // nel ciclo mi salvo le coordinate 
    const [r, c] = cell.id.split('-').map(Number);
    // let coords = cell.id.split('-');
    // let r = parseInt(coords[0]);
    // let c = parseInt(coords[1]);

    if(board[r][c] != '-'){
      cell.classList.add('cell-start');
    }
  });
  
  // la variabile "cella selezionata" torna ad essere nulla 
  cellSelected.value = null;

   // Disabilita nuovamente i bottoni numerici
  disableAllButtons();

  //togli gli stili all'ultimo bottone selezionato
  if(numSelected.value != null){
    numSelected.value.style.cssText = "";
  }
  numSelected.value = null;

}

// Disabilita tutti i bottoni numerici
function disableAllButtons(){
  digitNumbers.forEach(button => {
    button.setAttribute('disabled', true);
    button.style.cssText = "";
  });
}

function selectCell(id){

  
  resetCellStyles();

  // valorizzo la variabile "cellSelected" con la cella selezionata
  cellSelected.value = document.getElementById(id);

  // console.log("cella selezionata", cellSelected.value);

  // se la cella è vuota aggiungo la classe "cell-selected" 
  if(cellSelected.value.innerText === ''){
    cellSelected.value.classList.add('cell-selected');

    //abilito i numeri
    enableNumber(id);
  }
  
  // se la cella selezionata è un errore aggiungo un bordo interno
  if(cellSelected.value.classList.contains('bg-danger')){
    cellSelected.value.style.cssText ="box-shadow: inset 0 0 0 3px rgb(87, 101, 233)";

    enableNumber(id);
  }
  
  //evidenzio righe e colonne della cella selezionata
  highlightRowCol(id);
  

}

// attiva i bottoni dei numeri 
function enableNumber(){

    // se la cella continene già un numero in partenza BLOCCA e non fai inserire altri numeri
  if(cellSelected.value.classList.contains('cell-start')){
    return;
  }

  // 1. abilito  tutti i bottoni
  digitNumbers.forEach(button => {
    button.removeAttribute('disabled');
  });

  // 2. disattivo quelli non disponibili
   // ciclo su tutti i contatori e disabilita i bottoni associati se il contatore ha raggiunto la lunghezza massima
  for (let i = 1; i <= 9; i++) {
    const counter = counters[i].value;
    if (counter === limit.value) {
      document.getElementById(i.toString()).setAttribute('disabled', true);
    }
  }

}

function insertNumber(number){

  // TODO: togliere gli stili evidenziati a tutti i numeri delle celle e al bottone con lo stesso numero

  // queste azioni le svolgo nel caso in cui si inserisce un numero sbagliato nella stessa cella più di una volta 
  // Rimuovi gli stili dalle celle
  allCells.forEach(cell => {
    cell.classList.remove('equal-number');
  });
  
  // Rimuovi gli stili dai bottoni
  digitNumbers.forEach(button => {
    button.style.cssText = "";
  });
  // ---------------------------------------------
  
  // valorizzo la variabile "numSelected" col bottone/numero selezionato
  numSelected.value = document.getElementById(number);

  // inserisci nella cella il numero selezionato 
  cellSelected.value.innerText = numSelected.value.id;

  // mi salvo le coordinate della cella cliccata e le divido in due variabili
  const [r, c] = cellSelected.value.id.split('-').map(Number);
  // let coords = cellSelected.value.id.split('-');
  // let r = parseInt(coords[0]); // riga della cella selezionata
  // let c = parseInt(coords[1]); // colonna della cella selezionata

  // se il numero che si vuole inserire corrisponde al numero giusto in quella posizione
  if(solution[r][c] == numSelected.value.id){
    console.log("Hai inserito il numero GIUSTO!!");

    //FIXME: se il numero è corretto mi fai la stessa cosa che succede quando clicco una cella start cioè:
    
    //1. mi disabiliti tutti i bottoni 
    disableAllButtons();

    //2. mi evidenzi gli stessi numeri nella #board
    highlightRowCol(cellSelected.value.id);

    // --------------------
    cellSelected.value.classList.remove('bg-danger');
    // aggiungo nuovamente la classe "cell-selected" se in precedenza è stato inserito un numero errato e quindi è stata tolta per agigungere la classe "bg-danger"
    cellSelected.value.classList.add('cell-selected');

    // evidenzia il bottone del numero corretto appena inserito
    numSelected.value.style.cssText = "background-color: lightgray; color: #00eef7";

    // incremeto il contatore di quel numero
    // eval(`counter${numSelected.value.id}.value++`);
    incrementCounter(parseInt(numSelected.value.id));

    correctCount.value++; // Incrementa solo se il numero inserito è corretto
    
    // Verifica se tutti i numeri sono stati inseriti correttamente
    if (correctCount.value === emptyCellCount) {
      gameOver();
    }

    // la variabile "cella selezionata" torna ad essere nulla 
    cellSelected.value = null;

  } else {

    //incremeto il contatore degli errori
    errors.value ++;
    // tolgo la classe "cell-selected" per aggiungere "bg-danger"
    cellSelected.value.classList.remove('cell-selected');
    cellSelected.value.classList.add('bg-danger');

    // aggiungo il bordo interno 
    cellSelected.value.style.cssText ="box-shadow: inset 0 0 0 3px rgb(87, 101, 233)";

    //2. mi evidenzi gli stessi numeri nella #board
    highlightRowCol(cellSelected.value.id);

    // animazione
    const numOfErrors = document.getElementById('errors');
    // aggiungo
    setTimeout(() => {
      numOfErrors.classList.add('enlarge');
    }, 50)
    // rimuovo
    setTimeout(() => {
      numOfErrors.classList.remove('enlarge');
    }, 1000)

  }

  
}

// questa funzione evidenzia tutta la riga, tutta la colonna e tutto il quadrato della cella selezionata e anche gli stessi numeri nelle altre celle
function highlightRowCol(id){

  // mi salvo le coordinate tramite l'id della cella selezionata
  const [rSelectedCell, cSelectedCell] = id.split('-').map(Number);
  // let coordsSelectedCell = id.split('-');
  // let rSelectedCell = parseInt(coordsSelectedCell[0]);
  // let cSelectedCell = parseInt(coordsSelectedCell[1]);

  
  // ciclo tutte le celle 
  allCells.forEach(cell => {
    const [r, c] = cell.id.split('-').map(Number);
    // let coords = cell.id.split('-');
    // let r = parseInt(coords[0]);
    // let c = parseInt(coords[1]);


    // evidenzio le celle della stessa riga e della stessa colonna alla cella selezionata
    if (r === rSelectedCell || c === cSelectedCell){
      cell.classList.add('row-col-selected');
      if(cell.classList.contains('cell-start')){
        // dato che le celle iniziali hanno già un background mi trovo costretto a cambiare classe per modificare il backgroud-color
        // TODO: trovare una soluzione alternativa per evidenzaire le celle iniziali
        cell.classList.remove('cell-start');
        cell.classList.add('cell-start-selected'); // questa cosa si riperquote anche nel reset resetCellStyles()
      }
    }

    // evidenzia anche il quadrato
    const inSameRow = r === rSelectedCell;
    const inSameCol = c === cSelectedCell;
    const inSameSquare =
      Math.floor(r / 3) === Math.floor(rSelectedCell / 3) &&
      Math.floor(c / 3) === Math.floor(cSelectedCell / 3);

    if (inSameRow || inSameCol || inSameSquare) {
      cell.classList.add('row-col-selected');
      if (cell.classList.contains('cell-start')) {
        cell.classList.remove('cell-start');
        cell.classList.add('cell-start-selected');
      }
    }

    // evidenzio tutti i numeri uguali
    // se la cella NON è vuota e se all'interno c'è lo stesso numero cliccato aggiungo la classe "equal-number"
    if(cellSelected.value.innerText !== '' && cell.innerText === cellSelected.value.innerText){
      cell.classList.add('equal-number');
    }

  }) // fine del ciclo di tutte le celle

    // mi evidenzia anche il bottone con quel numero
  digitNumbers.forEach(button => {
    if(button.id === cellSelected.value.innerText){
      button.style.cssText = "background-color: lightgray; color: #00eef7";
    }
  });
}


function printNumber(r, c){
  if(board[(r - 1)][(c - 1)] != "-"){
    return board[(r - 1)][(c - 1)]
  }
}

function gameOver() {
  console.log("%cHAI VINTO!!!", 'color: #1df700');


  const game = document.getElementById('game');
  game.classList.add('fade-out');

  setTimeout(() => {
    
    isGameOver.value = true;

  }, 2000);
}


// Rimuovi l'event listener quando il componente viene distrutto per evitare memory leak
onUnmounted(() => {
  document.removeEventListener('click', handleClickOutsideCell);
});

</script>

<template>
  <main>
    <div class="container text-center pt-3">

      <div v-if="isGameOver" id="end-game" class="fade-in py-4">
        <h1>Eccellente! Hai finito il sudoku con successo</h1>
      </div>

      <div v-else id="game">
        
        <h2>SUDOKU <span class="text-info">Livello: facile</span></h2>
      
        <div class="fw-bold fs-3 ">Errori: 
          <span 
            id="errors"
            :class="{
              'text-danger': errors > 0
              }">
            {{errors}}
          </span>
        </div>
      
          <div id="board" class="mx-auto mb-3 border border-white border-3">
            <div v-for="c in limit" :key="c">
              <div v-for="r in limit" :key="r">
    
                <div
                  :id="((r - 1) + '-' + (c - 1))"
                  class="cell"
                  :class="{ 
                    'cell-start': (board[(r - 1)][(c - 1)] != '-'), 
                    'horizontal-line': (r == 3 || r == 6), 
                    'vertical-line': (c == 3 || c == 6)
                  }"
                  @click="selectCell((r - 1) + '-' + (c - 1))"
                  > <!-- al click passo l'id come parametro-->
                  {{ printNumber(r,c) }}
                </div>
    
              </div>
            </div>
          </div>
      
          <div id="digits" class="mx-auto">
            <div class="row-ac d-flex px-3">
              
              <div class="col d-flex flex-wrap justify-content-center px-1">
              <!-- <div class="col col-10 d-flex flex-wrap"> -->
                <!--TODO: per la matita togliere i commenti -->
                
                <button
                  v-for="i in limit"
                  :key="i"
                  :id="i"
                  type="button"
                  class="number btn btn-outline-light" 
                  disabled 
                  @click="insertNumber(i)">
                  <!-- data-bs-toggle="button" -->
                  {{i}}
                </button>
    
                <!-- <button type="button" id="delete" class="btn btn-outline-light p-0 px-1" disabled>
                  <i class="fa-solid fa-delete-left"></i>
                </button> -->

              </div>
    
              <!-- TODO: toggle da btn-light a btn-warning -->
              <!-- <div class="col col-2 p-0 rounded">
                <span class="btn btn-outline-light h-100 d-flex align-items-center justify-content-center">
                  <i class="fa-solid fa-pencil"></i>
                </span>
              </div> -->

            </div>
          </div>

      </div>

    </div>

  </main>

</template>

<style lang="scss" scoped>
main{
  min-height: 100vh;
  
  .container{

  // TODO: rendere tutto più smooth: l'inerimento di un numero, evidenziare righe, colonne, stessi numeri uguali, tutto più fluido e non diretto come ora
    
    #board{
      box-sizing: content-box;
      width: 328px;
      // height: 328px;
      aspect-ratio: 1 / 1;
      
      display: flex;
      flex-wrap: wrap;

      .cell{
        box-sizing: content-box;
        width: 34px;
        // height: 34px;
        aspect-ratio: 1 / 1;
        border: 1px solid rgb(136, 136, 136);
      }

      .cell-start{
        background-color: rgb(90, 90, 90);
      }
  
      .cell-selected{
        background-color: white !important;
        color: black;
        box-shadow: inset 0 0 0 3px rgb(87, 101, 233); /* Bordo interno */
      }

      .equal-number{
        color: #00eef7;
      }
  
      .row-col-selected{
        background-color: rgba(114, 233, 241, 0.3);
      }
  
      .cell-start-selected{
        background-color: rgba(51, 162, 170, 0.8);
      }
  
      .horizontal-line {
        border-bottom: 3px solid white;
      }
  
      .vertical-line {
        border-right: 3px solid white;
      }
    }


    .number, .cell{
      font-size: 1.4rem;
      font-weight: bold;

      display: flex;
      justify-content: center;
      align-items: center;
    }


    #digits{

      .number{
        width: 46px;
        // height: 46px;
        aspect-ratio: 1 / 1;
        margin: 2px;
      }

      #delete{
        font-size: 2rem;
        margin: 2px;
      }
    }

    .fade-in {
      animation: fade 1s;
      animation-fill-mode: forwards;
    }

    .fade-out {
      animation: fade 2s;
      animation-fill-mode: forwards;
      animation-direction: reverse;
    }

    @keyframes fade {
      0%   { opacity: 0; }
      100% { opacity: 1; }
    }

    .enlarge{
      display: inline-block;
      animation: enlarge 1s;
    }

    @keyframes enlarge {
      0% {transform: scale(1);}
      50% {transform: scale(1.4);}
      100% {transform: scale(1);}
      
    }

  }
}
</style>