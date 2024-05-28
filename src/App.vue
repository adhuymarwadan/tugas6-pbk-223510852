<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input
        type="text"
        v-model="form.title"
        placeholder="Title"
        class="input"
      /><br />
      <textarea
        v-model="form.content"
        placeholder="Content"
        class="textarea"
      ></textarea
      ><br />
      <button type="submit" class="btn">Save</button>
    </form>
    <button @click="load" class="btn-load">Load Articles</button>
    <div class="article-list">
      <div v-for="article in articles" :key="article.id" class="article-item">
        <div class="article-header">
          <h3>{{ article.title }}</h3>
          <div class="article-buttons">
            <button @click="edit(article)" class="btn-edit">Edit</button>
            <button @click="remove(article.id)" class="btn-delete">
              Delete
            </button>
          </div>
        </div>
        <p>{{ article.content }}</p>
      </div>
    </div>
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
          form.id = response.data.id;
        }
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

    onMounted(load);

    return { form, articles, save, edit, remove };
  },
};
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.form {
  width: 100%;
  padding: 20px;
  margin-bottom: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  background-color: rgba(255, 255, 255, 0.5);
}

.input,
.textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
  background-color: rgba(255, 255, 255, 0.5);
  color: #333;
}

.btn,
.btn-edit,
.btn-delete,
.btn-load {
  display: inline-block;
  padding: 10px 15px;
  margin: 5px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  text-align: center;
}

.btn {
  background-color: #28a745;
  color: #fff;
}

.btn-edit {
  background-color: #007bff;
  color: #fff;
}

.btn-delete {
  background-color: #dc3545;
  color: #fff;
}

.btn-load {
  background-color: #ffc107;
  color: #fff;
}

.article-list {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.article-item {
  background-color: rgba(255, 255, 255, 0.5);
  padding: 20px;
  flex: 1 1 calc(33.333% - 20px);
  border: 1px solid #ddd;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  box-sizing: border-box;
  color: #333;
}

.article-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.article-buttons {
  display: flex;
  gap: 10px;
}
</style>
