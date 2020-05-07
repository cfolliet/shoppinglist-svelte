<script>
  import { onMount } from "svelte";
  let displaySettings = false;
  let accountKey = "";
  let keyword = "";
  let displayAll = false;
  let items = [];
  $: isVisible = function(item) {
    if (keyword.length > 0) {
      return (
        item.section ||
        item.name.toLowerCase().indexOf(keyword.toLowerCase()) > -1
      );
    }

    if (!displayAll) {
      if (!item.section) {
        return !item.checked;
      }

      const index = items.indexOf(item);
      const nextSectionIndex = items.findIndex(
        (i2, index2) => i2.section && index2 > index
      );
      const result = items.some((i3, index3) => {
        return (
          !i3.checked &&
          index3 > index &&
          (index3 < nextSectionIndex || nextSectionIndex == -1)
        );
      });

      return result;
    }

    return true;
  };
  function add(e) {
    let name = this.keyword.trim();
    let isSection = false;
    if (name.startsWith("#")) {
      name = name.substring(1);
      isSection = true;
    }

    const li = e.target.closest("li");
    const sectionIndex = li.getAttribute("data-index");
    let nextSectionIndex = this.items.findIndex(
      (i, index) => i.section && index > sectionIndex
    );
    nextSectionIndex =
      nextSectionIndex > 0 ? nextSectionIndex : this.items.length;
    this.items.splice(nextSectionIndex, 0, {
      name: name,
      checked: false,
      section: isSection
    });
    save(this.accountKey, this.items);
    this.clear();
  }
  function remove(index) {
    this.items.splice(index, 1);
    save(this.accountKey, this.items);
  }
  function clear() {
    this.keyword = "";
    document
      .querySelector(".action-bar>.mdl-textfield")
      .MaterialTextfield.change();
  }

  const firebaseConfig = {
    apiKey: "AIzaSyA3smcaTh9oWfJs0RP8qnviK1lwfUWDir8",
    authDomain: "shoppinglist2-eb98b.firebaseapp.com",
    databaseURL: "https://shoppinglist2-eb98b.firebaseio.com",
    projectId: "shoppinglist2-eb98b",
    storageBucket: "shoppinglist2-eb98b.appspot.com",
    messagingSenderId: "782907887913",
    appId: "1:782907887913:web:f606afcae9665f335fa738"
  };

  const collectionId = "lists";
  const docId = "test";

  onMount(() => {
    window.addEventListener("load", event => {
      firebase.initializeApp(firebaseConfig);
      const db = firebase.firestore();
      const docRef = db.collection(collectionId).doc(docId);
      docRef.onSnapshot(function(doc) {
        if (doc.exists) {
          const value = doc.data().value || [
            { name: "section 1", section: true },
            { name: "item" }
          ];
          items = value;
        } else {
          console.error("No such document!");
        }
      });
    });
  });
</script>

<style>
  body {
    background-color: #fafafa;
  }

  #app {
    padding: 25px;
  }

  .settings {
    top: 20px;
    right: 20px;
    position: absolute;
  }

  .mdl-list {
    padding: 0;
    margin-bottom: 40px;
  }

  .section {
    color: #4a5a6a;
    text-transform: uppercase;
    font-size: 10px;
    font-weight: bold;
  }

  .item {
    font-size: 20px;
    padding-left: 40px;
    white-space: nowrap;
  }

  .checked .item-content {
    opacity: 0.5;
    text-decoration: line-through;
  }

  .check-indicator {
    position: absolute;
    left: 60px;
  }

  .display-check .check-indicator,
  .display-check .check-indicator-uncheck,
  .checked.display-check .check-indicator-checked {
    display: inline;
  }

  .check-indicator-uncheck,
  .check-indicator-checked,
  .checked .check-indicator-uncheck {
    display: none;
  }

  .button-sort {
    color: rgb(63, 81, 181);
  }

  .sort-hover {
    border-bottom: rgb(63, 81, 181) 1px solid;
  }

  .button-delete {
    position: absolute;
    right: 20px;
  }

  .action-bar {
    position: fixed;
    bottom: 0px;
    background-color: #fafafa;
  }
</style>

<svelte:head>
  <script src="https://www.gstatic.com/firebasejs/7.14.1/firebase-app.js">

  </script>
  <script src="https://www.gstatic.com/firebasejs/7.14.1/firebase-firestore.js">

  </script>
</svelte:head>
<ul class="mdl-list">
  {#each items as item, i}
    {#if isVisible(item)}
      <li
        data-index="items.indexOf(item)"
        class="mdl-list__item"
        class:section={item.section}
        class:item={!item.section}
        class:checked={item.checked}>
        <span class="mdl-list__item-primary-content">
          {#if !item.section}
            <div class="check-indicator">
              <i class="material-icons check-indicator-uncheck">
                check_box_outline_blank
              </i>
              <i class="material-icons check-indicator-checked">check_box</i>
            </div>
          {/if}
          <div class="item-content">
            {#if item.hover}
              <i class="mdl-icon-toggle__label material-icons button-sort">
                swap_vert
              </i>
            {/if}
            <span>{item.name}</span>
            {#if item.section && keyword.length}
              <button
                on:click={add}
                class="mdl-button mdl-js-button mdl-button--fab
                mdl-button--mini-fab mdl-button--colored">
                <i class="material-icons">add</i>
              </button>
            {/if}
            {#if item.hover}
              <button
                on:click={remove(items.indexOf(item))}
                class="mdl-button mdl-js-button mdl-button--icon
                mdl-button--colored button-delete">
                <i class="material-icons">delete</i>
              </button>
            {/if}
          </div>
        </span>
      </li>
    {/if}
  {/each}
  <div class="action-bar">
    <div class="mdl-textfield mdl-js-textfield">
      <input
        class="mdl-textfield__input"
        type="text"
        id="keyword"
        bind:value={keyword} />
      <label class="mdl-textfield__label" for="keyword">
        Item or #Section...
      </label>
    </div>
    <button on:click={clear} class="mdl-button mdl-js-button mdl-button--icon">
      <i class="material-icons">clear</i>
    </button>
    <label
      class="mdl-icon-toggle mdl-js-icon-toggle mdl-js-ripple-effect"
      for="visibility">
      <input
        type="checkbox"
        id="visibility"
        class="mdl-icon-toggle__input"
        bind:checked={displayAll} />
      <i class="mdl-icon-toggle__label material-icons">visibility</i>
    </label>
  </div>
</ul>
