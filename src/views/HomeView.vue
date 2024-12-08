<template>
  <div class="container">
    <div class="actions">
      <button @click="goBack">
        <img
          class="actions-icon"
          src="https://img.icons8.com/?size=100&id=85099&format=png&color=ffffff"
          alt=""
        />
      </button>
      <button @click="goNext">
        <img
          class="actions-icon"
          src="https://img.icons8.com/?size=100&id=86520&format=png&color=FFFFFF"
          alt=""
        />
      </button>
      <button @click="changeTag('h2')">
        <img
          class="actions-icon"
          src="https://img.icons8.com/?size=100&id=egrmUicA9GdW&format=png&color=FFFFFF"
          alt=""
        />
      </button>
      <button @click="changeTag('p')">
        <img
          class="actions-icon"
          src="https://img.icons8.com/?size=100&id=78999&format=png&color=FFFFFF"
          alt=""
        />
      </button>
      <input type="text" v-model="imgUrl" />
      <button @click="insertImageAtCursor()">
        <img
          class="actions-icon"
          src="https://img.icons8.com/?size=100&id=78741&format=png&color=FFFFFF"
          alt=""
        />
      </button>
      <button @click="copyData()">Скопировать HTML</button>
    </div>
    <div
      class="content"
      id="content"
      contenteditable="true"
      @input="saveState"
    ></div>
  </div>
</template>
<script setup>
import { onMounted, reactive, ref } from "vue";

let imgUrl = ref("");

const undoStack = reactive([]);

const indexRec = ref(0);

function saveState() {
  const editor = document.getElementById("content").innerHTML;
  undoStack.push(editor);
  indexRec.value = undoStack.length;
}

const goBack = () => {
  if (indexRec.value === 0) return;
  indexRec.value = indexRec.value - 1;
  document.getElementById("content").innerHTML = undoStack[indexRec.value];
};

const goNext = () => {
  if (undoStack.length === indexRec.value + 1) return;
  indexRec.value += 1;
  console.log(indexRec.value);
  document.getElementById("content").innerHTML = undoStack[indexRec.value];
};

const copyData = () => {
  navigator.clipboard.writeText(document.getElementById("content").innerHTML);
};

const changeTag = (tag) => {
  const select = window.getSelection();
  if (!select.rangeCount) return;

  const range = select.getRangeAt(0);

  let parentElement = range.commonAncestorContainer;
  if (parentElement.nodeType === 3) {
    parentElement = parentElement.parentElement;
  }

  if (parentElement.classList.contains("content")) {
    const selectedText = range.toString();
    if (selectedText.trim() !== "") {
      const newElement = document.createElement(tag);
      newElement.textContent = selectedText;

      range.deleteContents();
      range.insertNode(newElement);
      saveState();
    }
  } else {
    const newElement = document.createElement(tag);
    newElement.innerHTML = parentElement.innerHTML;

    parentElement.replaceWith(newElement);
    saveState();
  }

  select.removeAllRanges();
};

const insertImageAtCursor = () => {
  const select = window.getSelection();
  if (!select.rangeCount) return;
  const range = select.getRangeAt(0);
  let parentElement = range.commonAncestorContainer;
  if (parentElement.classList.contains("actions")) {
    alert("Кликните в рабочую область");
    return;
  }

  const img = document.createElement("img");
  img.src = imgUrl.value;
  img.style.maxWidth = "100%";

  range.deleteContents();
  range.insertNode(img);

  range.setStartAfter(img);
  range.setEndAfter(img);

  select.removeAllRanges();
  select.addRange(range);
  saveState();
};
onMounted(() => {
  saveState();
});
</script>
<style lang="scss">
.actions {
  display: flex;
  gap: 10px;
  .actions-icon {
    width: 35px;
  }
}
.content {
  margin-top: 50px;
  min-height: 100vh;
}
</style>
