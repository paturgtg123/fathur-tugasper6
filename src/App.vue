<template>
  <div>
    <div class="input">
      <h1>Tambahkan List Anda</h1>
      <form @submit.prevent="save">
        <input type="text" v-model="form.title" placeholder="Title" /><br />
        <textarea v-model="form.content" placeholder="Content"></textarea><br />
        <button type="submit">Save</button>
      </form>
    </div>

    <div class="article-list">
      <div v-for="article in articles" :key="article.id" class="article-item">
        <strong>{{ article.title }}</strong
        ><br />
        <span>{{ article.content }}</span
        ><br />
        <button @click="edit(article)">Edit</button>
        <button @click="deleteArticle(article.id)">Delete</button>
      </div>
    </div>

    <button @click="load">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from "vue";
import axios from "axios";

export default {
  setup() {
    const form = reactive({
      id: null,
      title: "",
      content: "",
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get("http://localhost:3000/articles");
        articles.value = response.data;
      } catch (error) {
        console.error("Error loading articles:", error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : "http://localhost:3000/articles";
        const method = form.id ? "put" : "post";
        const response = await axios[method](url, form);
        if (form.id) {
          const index = articles.value.findIndex(
            (article) => article.id === form.id
          );
          if (index !== -1) {
            articles.value[index] = response.data;
          }
        } else {
          articles.value.push(response.data);
        }
        form.id = null;
        form.title = "";
        form.content = "";
      } catch (error) {
        console.error("Error saving article:", error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter((article) => article.id !== id);
      } catch (error) {
        console.error("Error deleting article:", error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    async function deleteArticle(id) {
      if (confirm("Are you sure you want to delete this article?")) {
        await remove(id);
      }
    }

    onMounted(load);

    return { form, articles, save, edit, deleteArticle };
  },
};
</script>

<style>
.input {
  text-align: center;
  color: aquamarine;
}
body {
  background-image: url(https://wallpaperaccess.com/full/1150158.jpg);
  background-repeat: no-repeat;
  background-size: cover;
  margin: 0;
  display: flex;
  place-items: center;
  min-width: 320px;
  min-height: 100vh;
}
.article-list {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}
.article-item {
  border: 1px solid #ddd;
  padding: 10px;
  border-radius: 4px;
  max-width: 200px;
}
</style>
