<template>
  <div
    v-loading="loading"
    element-loading-text="Loading..."
    element-loading-spinner="el-icon-loading"
  >
    <div v-if="recipe" class="details-container">
      <div class="top-action-buttons">
        <el-button type="text" @click="goBack" style="margin-bottom: 20px">
          ← Back to Dashboard
        </el-button>

        <el-button type="primary" @click="goToEdit" style="float: right"
          >Edit</el-button
        >
        <el-button
          type="danger"
          style="float: right"
          @click="confirmDelete"
          :loading="deleting"
          :disabled="deleting"
        >
          Delete
        </el-button>
      </div>

      <h2>{{ recipe.title }}</h2>

      <div class="details-flex">
        <div class="details-image">
          <img :src="getImageUrl(recipe.imageUrl)" alt="Recipe Image" />
        </div>

        <div class="details-info">
          <el-descriptions :column="1" border>
            <el-descriptions-item label="Difficulty">
              {{ recipe.difficulty }}
            </el-descriptions-item>

            <el-descriptions-item label="Creator">
              {{ recipe.creatorName }}
            </el-descriptions-item>

            <el-descriptions-item label="Created On">
              {{ recipe.createdDate.slice(0, 10) }}
            </el-descriptions-item>

            <el-descriptions-item label="Ingredients">
              <el-tag
                v-for="(ingredient, index) in recipe.ingredients"
                :key="index"
                type="success"
                class="ingredient-tag"
              >
                {{ ingredient.name }}
              </el-tag>
            </el-descriptions-item>

            <el-descriptions-item label="Instructions">
              <div style="white-space: pre-wrap">
                {{ recipe.instruction || "No instructions provided." }}
              </div>
            </el-descriptions-item>
          </el-descriptions>
        </div>
      </div>
    </div>

    <div v-else class="empty-message">
      <el-empty description="Recipe not found or still loading..." />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useRecipeStore } from "../stores/recipeStore";
import { ElMessageBox, ElMessage } from "element-plus";

const route = useRoute();
const router = useRouter();
const recipeStore = useRecipeStore();
const loading = ref(true);
const deleting = ref(false);

const recipe = ref(null);

const loadRecipe = async () => {
  loading.value = true;
  const id = route.params.id;
  const result = await recipeStore.fetchRecipeById(id);
  recipe.value = result;
  loading.value = false;
  console.log(recipe.value);
};

const getImageUrl = (relativePath) => {
  console.log(relativePath);
  return relativePath && relativePath !== "/uploads/null" && relativePath !== ""
    ? `https://back-end-oo5f.onrender.com${relativePath}` //contention
    : "/images/placeholder-image.jpg"; // fallback image
};

const confirmDelete = async () => {
  try {
    await ElMessageBox.confirm(
      "Are you sure you want to delete this recipe?",
      "Delete Confirmation",
      {
        confirmButtonText: "Delete",
        cancelButtonText: "Cancel",
        type: "warning",
      }
    );
    deleting.value = true;
    await recipeStore.deleteRecipe(route.params.id);
    ElMessage.success("Recipe deleted successfully");

    // Wait briefly to show success message, then redirect
    setTimeout(() => {
      router.push("/");
    }, 500);
  } catch (error) {
    if (error !== "cancel") {
      ElMessage.error("Failed to delete recipe");
    }
  } finally {
    deleting.value = false;
  }
};

const goBack = () => {
  router.push("/");
};

const goToEdit = () => {
  router.push(`/recipes/${route.params.id}/edit`);
};

onMounted(loadRecipe);
</script>

<style scoped>
.details-container {
  padding: 24px;
  margin: 24px;
}

.ingredient-tag {
  margin-right: 8px;
  margin-top: 5px;
}

.empty-message {
  margin: 40px auto;
  text-align: center;
}

.details-flex {
  display: flex;
  gap: 32px;
  align-items: flex-start;
  margin-top: 20px;
}

.details-image img {
  width: 400px;
  height: 300px;
  object-fit: cover;
  object-position: center;
  border-radius: 8px;
  background-color: #f4f4f4;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
}

.details-info {
  flex: 1;
}

h2 {
  font-size: 28px;
  font-weight: 600;
  margin-bottom: 16px;
}

.top-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}
.action-right-buttons {
  display: flex;
  align-items: center;
}
</style>
