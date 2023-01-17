<script>
  import { onMount, tick } from "svelte";

  let records = [];
  let showConfirmation = false;
  let currentRecord;
  let editingRecord;

  function sortByDate(a, b) {
    return new Date(b.date) - new Date(a.date);
  }

  function sortRecords(sortBy) {
    let cat = document.getElementById("category-filter").value;
    records = JSON.parse(localStorage.getItem("records")) || [];
    if (cat != "all") {
      records = JSON.parse(localStorage.getItem("filtered-records")) || [];
    }
    if (sortBy === "date") {
      records.sort((a, b) => new Date(b.date) - new Date(a.date));
    } else if (sortBy === "category") {
      records.sort((a, b) => a.category.localeCompare(b.category));
    } else if (sortBy === "amount") {
      records.sort((a, b) => b.amount - a.amount);
    }
    if (cat != "all") {
      localStorage.setItem("filtered-records", JSON.stringify(records));
      records = JSON.parse(localStorage.getItem("filtered-records")) || [];
    } else {
      localStorage.setItem("records", JSON.stringify(records));
      records = JSON.parse(localStorage.getItem("records")) || [];
    }
    tick();
  }

  function filterByCategory(category) {
    records = JSON.parse(localStorage.getItem("records")) || [];
    if (category === "all") {
      // get select id element
      let select = document.getElementById("filterselector").value;
      sortRecords(select);
      return;
    }
    let filteredRecords = records.filter(
      (record) => record.category === category
    );
    localStorage.setItem("filtered-records", JSON.stringify(filteredRecords));
    records = JSON.parse(localStorage.getItem("filtered-records")) || [];
  }

  onMount(async () => {
    records = JSON.parse(localStorage.getItem("records")) || [];
    records.sort(sortByDate);
    if (localStorage.getItem("dark-mode") === "true") {
      window.document.body.classList.add("dark-mode");
    }
  });

  let newRecord = {
    amount: 0,
    category: "",
    date: new Date().toISOString().slice(0, 10),
    isIncome: false,
    details: "",
  };

  function createRecord() {
    records = JSON.parse(localStorage.getItem("records")) || [];
    if (newRecord.amount > 0) {
      showConfirmation = true;
      // Enregistrement des données dans le stockage local
      records = JSON.parse(localStorage.getItem("records")) || [];
      records.push(newRecord);
      records.sort(sortByDate);
      localStorage.setItem("records", JSON.stringify(records));
      setTimeout(() => {
        showConfirmation = false;
      }, 1950);
      // Réinitialisation du formulaire
      newRecord = {
        amount: 0,
        category: "",
        date: new Date().toISOString().slice(0, 10),
        isIncome: newRecord.isIncome,
        details: "",
      };
    } else {
      alert(
        "Le montant doit être supérieur à zéro pour créer un enregistrement."
      );
    }
  }

  function editRecord(record) {
    records = JSON.parse(localStorage.getItem("records")) || [];
    if (editingRecord) {
      editingRecord = null;
      return;
    }
    currentRecord = { ...record };
    editingRecord = record;
  }

  function updateRecord() {
    records = JSON.parse(localStorage.getItem("records")) || [];
    if (editingRecord) {
      // Mise à jour de l'enregistrement dans le tableau "records"
      let index = records.indexOf(editingRecord);
      records.splice(index, 1, currentRecord);
      // Mise à jour du stockage local
      localStorage.setItem("records", JSON.stringify(records));
      editingRecord = null;
      records = JSON.parse(localStorage.getItem("records")) || [];
      tick();
    }
  }

  function cancelEdit() {
    editingRecord = null;
  }

  function deleteRecord(record) {
    records = JSON.parse(localStorage.getItem("records")) || [];
    if (confirm("Etes-vous sur de vouloir supprimer cet enregistrement?")) {
      // Suppression de l'enregistrement dans le tableau "records"
      let index = records.indexOf(record);
      records.splice(index, 1);
      // Mise à jour du stockage local
      localStorage.setItem("records", JSON.stringify(records));
      records = JSON.parse(localStorage.getItem("records")) || [];
      tick();
    }
  }
</script>

<svelte:head>
  <title>MoneyManager</title>
</svelte:head>
{#if showConfirmation}
  <div class="confirmation-message">Enregistrement créé avec succès!</div>
{/if}
<form class="entryform" on:submit|preventDefault={createRecord}>
  <label for="amount">Montant :</label>
  <input type="number" min="0" id="amount" bind:value={newRecord.amount} />
  <br />
  <label for="category">Catégorie :</label>
  <select id="category" bind:value={newRecord.category}>
    <option value="alimentation">Alimentation</option>
    <option value="logement">Logement</option>
    <option value="transport">Transport</option>
    <option value="loisirs">Loisirs</option>
    <option value="sante">Santé</option>
    <option value="habillement">Habillement</option>
    <option value="communication">Communication</option>
    <option value="education">Education</option>
    <option value="investissement">Investissement</option>
    <option value="autres">Autres</option>
  </select>
  <br />
  <label for="date">Date :</label>
  <input type="date" id="date" bind:value={newRecord.date} />
  <br />
  <label for="isIncome">Type:</label>
  <select id="isIncome" bind:value={newRecord.isIncome}>
    <option value={false}>Dépense</option>
    <option value={true}>Revenu</option>
  </select>
  <br />
  <label for="details">Détails:</label>
  <textarea id="details" maxlength="200" bind:value={newRecord.details} />
  <br />
  <button class="submitbtn" type="submit">Créer</button>
</form>
<div class="spacing">&nbsp;</div>
<div>
  <h2>Records</h2>
  <p>Nombre de transactions : {records.length}</p>
  <div>
    <select
      id="filterselector"
      on:change={(event) => sortRecords(event.target.value)}
    >
      <option value="date">Date</option>
      <option value="category">Catégorie</option>
      <option value="amount">Montant</option>
    </select>
    <select
      id="category-filter"
      on:change={(event) => filterByCategory(event.target.value)}
    >
      <option value="all">Toutes les catégories</option>
      <option value="alimentation">Alimentation</option>
      <option value="logement">Logement</option>
      <option value="transport">Transport</option>
      <option value="loisirs">Loisirs</option>
      <option value="sante">Santé</option>
      <option value="habillement">Habillement</option>
      <option value="communication">Communication</option>
      <option value="education">Education</option>
      <option value="investissement">Investissement</option>
      <option value="autres">Autres</option>
      <!-- autres options -->
    </select>
  </div>
</div>
<table>
  <tr>
    <th>Montant</th>
    <th>Catégorie</th>
    <th>Date</th>
    <th>Type</th>
    <th style="width:300px">Détails</th>
    <th>Actions</th>
  </tr>
  {#each records as record}
    <tr
      class:borderincome={record.isIncome}
      class:borderexpense={!record.isIncome}
      style="max-height:200px;height:70px;min-height:50px"
    >
      <td class:income={record.isIncome} class:expense={!record.isIncome}>
        {record.isIncome ? "+" : "-"}
        {record.amount}
      </td>
      <td>{record.category}</td>
      <td>{record.date}</td>
      <td>{record.isIncome ? "Revenu" : "Dépense"}</td>
      <td style="max-width:200px;width:200px;min-width:200px"
        >{record.details}</td
      >
      <td>
        <button class="editbtn" on:click={() => editRecord(record)}>Edit</button
        >
        <button class="supprbtn" on:click={() => deleteRecord(record)}
          >Delete</button
        >
      </td>
    </tr>
    {#if editingRecord === record}
      <tr>
        <td colspan="5">
          <form class="edit-form" on:submit|preventDefault={updateRecord}>
            <label for="amount">Montant :</label>
            <input
              type="number"
              id="amount"
              bind:value={currentRecord.amount}
            />

            <label for="category">Catégorie :</label>
            <select id="category" bind:value={currentRecord.category}>
              <option value="alimentation">Alimentation</option>
              <option value="logement">Logement</option>
              <option value="transport">Transport</option>
              <option value="loisirs">Loisirs</option>
              <option value="sante">Santé</option>
              <option value="habillement">Habillement</option>
              <option value="communication">Communication</option>
              <option value="education">Education</option>
              <option value="investissement">Investissement</option>
              <option value="autres">Autres</option>
              <!-- autres options -->
            </select>

            <label for="date">Date :</label>
            <input type="date" id="date" bind:value={currentRecord.date} />

            <label for="isIncome">Type:</label>
            <select id="isIncome" bind:value={currentRecord.isIncome}>
              <option value={false}>Dépense</option>
              <option value={true}>Revenu</option>
            </select>

            <label for="details">Détails :</label>
            <textarea
              id="details"
              maxlength="200"
              bind:value={currentRecord.details}
            />

            <div>
              <button class="savebtn" type="submit" on:click={updateRecord}
                >Enregistrer</button
              >
              <button class="cancelbtn" type="button" on:click={cancelEdit}
                >Annuler</button
              >
            </div>
          </form>
        </td>
      </tr>
    {/if}
  {/each}
</table>

<style>
  :global(body) {
    background-color: #afa280;
    color: #424242;
    transition: background-color 0.3s;
  }
  :global(body.dark-mode) {
    background-color: #1d3040;
    color: #4d8585;
  }
  .income {
    font-weight: bold;
    color: rgb(1, 126, 1);
  }

  .expense {
    font-weight: bold;
    color: rgb(177, 2, 2);
  }

  .submitbtn {
    margin-right: 10px;
    margin-top: 10px;
  }

  #isIncome {
    max-width: 200px;
  }

  textarea {
    background-color: rgba(255, 255, 255, 0.1);
    padding: 1rem;
    margin-left: 10px;
    margin-right: 10px;
    border-radius: 0.5rem;
    border: none;
    box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.2);
    transition: all 0.3s ease-in-out;
    font-size: 1rem;
    resize: none;
    width: 75%;
    height: 3rem;
    color: black;
  }

  textarea:focus {
    background-color: rgba(255, 255, 255, 0.2);
    outline: none;
  }

  .spacing {
    height: 450px;
  }

  .entryform {
    position: fixed;
    display: inline;
    text-align: right;
    background-color: rgba(255, 255, 255, 0.9);
    box-shadow: 0px 4px 8px 0px rgba(0, 0, 0, 0.2);
    border-radius: 10px;
    padding: 20px;
    margin-right: 4vw;
    padding-left: 20vw;
  }

  .entryform label {
    position: absolute;
    left: 0px;
  }
  table {
    display: flex;
    flex-direction: column;
    background-color: rgba(255, 255, 255, 0.8);
    border-radius: 10px;
    box-shadow: 0px 5px 20px rgba(0, 0, 0, 0.1);
    overflow: hidden;
  }

  th,
  td {
    padding: 1rem;
    text-align: center;
  }

  td {
    display: inline-block;
  }

  th {
    background-color: rgba(0, 0, 0, 0.02);
    border-radius: 10px;
    font-weight: bold;
  }
  input,
  select {
    font-size: 1.2em;
    margin: 10px;
    padding: 10px;
    border-radius: 5px;
    background-color: rgba(255, 255, 255, 0.8);
    box-shadow: 0px 2px 4px 0px rgba(0, 0, 0, 0.1);
  }

  label {
    width: 100px;
    text-align: right;
    padding-right: 20px;
  }

  input,
  select {
    flex-grow: 1;
    border: none;
  }

  button[type="submit"] {
    background-color: #4caf50;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1.2em;
    margin-left: auto;
    box-shadow: 0px 2px 4px 0px rgba(0, 0, 0, 0.1);
  }

  .confirmation-message {
    background-color: rgba(0, 128, 0, 0.8);
    color: white;
    padding: 1rem;
    border-radius: 10px;
    text-align: center;
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    animation: fadeInOut 2s ease-in-out;
  }

  .editbtn {
    background-color: #4caf50; /* vert */
    color: white;
    padding: 12px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
    margin-right: 8px;
    transition: all 0.3s ease-in-out;
  }

  .editbtn:hover {
    background-color: #3e8e41;
  }

  .supprbtn {
    background-color: #c5372d; /* rouge */
    color: white;
    padding: 12px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
    margin-right: 8px;
    transition: all 0.3s ease-in-out;
  }

  /* Masque le formulaire d'édition par défaut */
  .edit-form {
    display: block;
  }

  /* Style pour les champs de formulaire */
  .edit-form input,
  .edit-form select {
    padding: 0.5rem;
    margin-bottom: 1rem;
    font-size: 1rem;
    border-radius: 5px;
    border: 1px solid #ccc;
  }

  /* Style pour les boutons de formulaire */
  .edit-form button {
    padding: 0.5rem 1rem;
    margin-right: 1rem;
    font-size: 1rem;
    border-radius: 5px;
    background-color: #007bff;
    color: #fff;
    border: none;
    cursor: pointer;
  }

  /* Style pour le bouton "Save" */
  .edit-form button.savebtn {
    background-color: #28a745;
  }

  /* Style pour le bouton "Cancel" */
  .edit-form button.cancelbtn {
    background-color: #dc3545;
  }

  .supprbtn:hover {
    background-color: #830e06;
  }

  @keyframes fadeInOut {
    0% {
      opacity: 0;
    }
    25% {
      opacity: 1;
    }
    75% {
      opacity: 1;
    }
    100% {
      opacity: 0;
    }
  }

  button[type="submit"]:hover {
    background-color: #3e8e41;
    box-shadow: 0px 4px 8px 0px rgba(0, 0, 0, 0.2);
  }
</style>
