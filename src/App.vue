<script setup>
import { ref, onMounted } from "vue";
import Quill from "quill";
import "quill/dist/quill.snow.css"; // import Quill styling

// Optional: add custom font & size support
// const Font = Quill.import("formats/font");
// Font.whitelist = ["poppins", "roboto", "inconsolata", "mirza"];
// Quill.register(Font, true);

const editor = ref(null);
const content = ref("");

onMounted(() => {
  const quill = new Quill(editor.value, {
    theme: "snow",
    placeholder: "Write something awesome...",
    modules: {
      toolbar: [
        [{ font: [] }],
        [{ header: [1, 2, 3, 4, 5, false] }], // font size
        ["bold", "italic", "underline", "strike"], // toggled buttons
        [{ color: [] }, { background: [] }], // text color & background
        [{ script: "sub" }, { script: "super" }], // superscript/subscript
        [{ header: 1 }, { header: 2 }], // headers
        [{ align: [] }], // text align
        [{ list: "ordered" }, { list: "bullet" }], // lists
        ["blockquote", "code-block"],
        ["link", "image", "video"],
        ["clean"], // remove formatting
      ],
    },
  });

  // Sync editor content with Vue ref
  quill.on("text-change", () => {
    content.value = quill.root.innerHTML;
  });
});
</script>

<template>
  <div class="mx-auto max-w-[1350px] mt-8 mb-4 px-6 font-Poppins">
    <div>
      <h2 class="text-2xl font-bold mb-4">Rich Text Editor with Quill.js</h2>
      <!-- Editor container -->
      <div ref="editor" style="height: 300px"></div>

      <!-- Preview -->
      <h3 class="mt-4">Output:</h3>
      <div v-html="content" class="border p-2 editor-content text-red-400"></div>
      <!-- <div class="border p-2 editor-content">{{ content }}</div> -->
    </div>
  </div>
</template>

<style>
.editor-content * {
  all: revert;
}
</style>
