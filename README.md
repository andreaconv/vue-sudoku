 La funzione `handleClickOutsideCell(event)` è un gestore degli eventi che viene chiamato ogni volta che si verifica un click sulla pagina. Questa funzione controlla se il click è avvenuto all'interno della cella appena selezionata o al di fuori di essa. Ecco una spiegazione dettagliata della funzione:

1. `event`: È l'oggetto evento generato quando si verifica un'azione, in questo caso un click sulla pagina.

2. `event.target`: È l'elemento DOM su cui è stato effettuato il click.

3. `event.target.closest('.cell')`: Questo metodo cerca l'elemento più vicino (compreso l'elemento stesso) che corrisponde al selettore specificato, in questo caso l'elemento con la classe .cell. Se l'elemento cliccato è una cella o si trova all'interno di una cella, event.target.closest('.cell') restituirà l'elemento stesso o la cella genitore più vicina.

4. `event.target.closest('.cell') === cellSelected.value`: Questo confronto controlla se l'elemento cliccato è la cella attualmente selezionata. cellSelected.value contiene un riferimento all'elemento della cella attualmente selezionata.

5. `isClickedInsideCell`: È una variabile booleana che indica se il click è avvenuto all'interno della cella selezionata o meno.

6. `!isClickedInsideCell`: Questo è un controllo condizionale che verifica se il click è avvenuto al di fuori della cella selezionata (cioè se isClickedInsideCell è false).

7. `resetCellStyles()`: Questa chiamata di funzione viene eseguita se il click è avvenuto al di fuori della cella selezionata. resetCellStyles() ripristina lo stato iniziale dello stile delle celle, rimuovendo la classe text-info da tutte le celle e rimuovendo anche la classe cell-selected dalla cella selezionata.

In sostanza, questa funzione garantisce che quando viene rilevato un click al di fuori della cella appena selezionata, lo stato delle celle viene ripristinato allo stato iniziale.

---

