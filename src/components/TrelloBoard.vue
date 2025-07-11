<template>
  <div class="trello-board bg-gradient bg-primary">
    <div class="d-flex justify-content-between align-items-center mb-4 px-3 pt-3">
      <h1 class="text-white mb-0">
        <i class="bi bi-kanban me-2"></i>
        To do list
      </h1>
      <button @click="addColumn" class="btn btn-light btn-sm">
        <i class="bi bi-plus-circle me-1"></i>
        Add Column
      </button>
    </div>
    <div class="board-content mx-3">
      <draggable 
        v-model="columns" 
        group="columns"
        class="d-flex gap-3 flex-nowrap"
        item-key="id"
        @start="drag = true"
        @end="drag = false"
      >
        <template #item="{ element: column }">
          <div class="column bg-white rounded shadow-sm">
            <div class="d-flex justify-content-between align-items-center p-3 border-bottom">
              <input 
                v-model="column.title" 
                class="form-control border-0 fw-bold"
                style="background: transparent;"
                @blur="updateColumn(column)"
              />
              <button @click="deleteColumn(column.id)" class="btn btn-outline-danger btn-sm">
                <i class="bi bi-trash"></i>
              </button>
            </div>
            <draggable 
              v-model="column.cards" 
              group="cards"
              class="p-3"
              style="min-height: 100px;"
              item-key="id"
              @start="drag = true"
              @end="drag = false"
            >
              <template #item="{ element: card }">
                <div class="card mb-2 shadow-sm" @click="editCard(card)" style="cursor: pointer;">
                  <div class="card-body p-3">
                    <h6 class="card-title mb-2">{{ card.title }}</h6>
                    <p v-if="card.description" class="card-text small text-muted mb-2">{{ card.description }}</p>
                    <div class="d-flex justify-content-between align-items-center">
                      <small class="text-muted">{{ formatDate(card.createdAt) }}</small>
                      <button @click.stop="deleteCard(column.id, card.id)" class="btn btn-outline-danger btn-sm">
                        <i class="bi bi-x"></i>
                      </button>
                    </div>
                  </div>
                </div>
              </template>
            </draggable>
            <div class="p-3">
              <button @click="addCard(column.id)" class="btn btn-outline-primary w-100">
                <i class="bi bi-plus-circle me-1"></i>
                Add Card
              </button>
            </div>
          </div>
        </template>
      </draggable>
    </div>
    <!-- Card Edit Modal giữ nguyên -->
    <div v-if="editingCard" class="modal fade show d-block" style="background: rgba(0,0,0,0.5);" @click="closeModal">
      <div class="modal-dialog" @click.stop>
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">
              <i class="bi bi-pencil-square me-2"></i>
              Edit Card
            </h5>
            <button @click="closeModal" class="btn-close"></button>
          </div>
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label">Title</label>
              <input v-model="editingCard.title" class="form-control" />
            </div>
            <div class="mb-3">
              <label class="form-label">Description</label>
              <textarea v-model="editingCard.description" class="form-control" rows="4"></textarea>
            </div>
          </div>
          <div class="modal-footer">
            <button @click="closeModal" class="btn btn-secondary">Cancel</button>
            <button @click="saveCard" class="btn btn-primary">
              <i class="bi bi-check-circle me-1"></i>
              Save
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useLocalStorage } from '@vueuse/core'
import draggable from 'vuedraggable'

const drag = ref(false)
const editingCard = ref(null)
const originalCard = ref(null)

// Sử dụng localStorage để lưu trữ dữ liệu
const columns = useLocalStorage('trello-columns', [
  {
    id: 1,
    title: 'To Do',
    cards: [
      { id: 1, title: 'Learn Vue 3', description: 'Study Vue 3 composition API', createdAt: new Date() },
      { id: 2, title: 'Build Trello Clone', description: 'Create a drag and drop board', createdAt: new Date() }
    ]
  },
  {
    id: 2,
    title: 'In Progress',
    cards: [
      { id: 3, title: 'Design UI', description: 'Create beautiful interface', createdAt: new Date() }
    ]
  },
  {
    id: 3,
    title: 'Done',
    cards: [
      { id: 4, title: 'Setup Project', description: 'Initialize Vue project', createdAt: new Date() }
    ]
  }
])

const addColumn = () => {
  const newColumn = {
    id: Date.now(),
    title: 'New Column',
    cards: []
  }
  columns.value.push(newColumn)
}

const deleteColumn = (columnId) => {
  const index = columns.value.findIndex(col => col.id === columnId)
  if (index > -1) {
    columns.value.splice(index, 1)
  }
}

const addCard = (columnId) => {
  const column = columns.value.find(col => col.id === columnId)
  if (column) {
    const newCard = {
      id: Date.now(),
      title: 'New Card',
      description: '',
      createdAt: new Date()
    }
    column.cards.push(newCard)
  }
}

const deleteCard = (columnId, cardId) => {
  const column = columns.value.find(col => col.id === columnId)
  if (column) {
    const index = column.cards.findIndex(card => card.id === cardId)
    if (index > -1) {
      column.cards.splice(index, 1)
    }
  }
}

const editCard = (card) => {
  editingCard.value = { ...card }
  originalCard.value = card
}

const saveCard = () => {
  if (originalCard.value && editingCard.value) {
    Object.assign(originalCard.value, editingCard.value)
  }
  closeModal()
}

const closeModal = () => {
  editingCard.value = null
  originalCard.value = null
}

const updateColumn = (column) => {
  // Column title is automatically updated via v-model
}

const formatDate = (date) => {
  return new Date(date).toLocaleDateString()
}
</script>

<style scoped>
.trello-board {
  min-height: 100vh;
  width: 100vw;
  min-width: 100vw;
  background: linear-gradient(135deg, #0d6efd 0%, #0a58ca 100%);
  padding: 10px 0 30px 0;
  overflow-x: hidden;
}

.board-content {
  width: 100vw;
  min-width: 100vw;
  position: relative;
  overflow-x: auto;
  padding-bottom: 20px;
  min-height: 80vh;
}

.d-flex.gap-3.flex-nowrap {
  flex-wrap: nowrap !important;
  justify-content: flex-start !important;
}

.column {
  transition: all 0.3s ease;
  min-width: 320px;
  max-width: 340px;
  margin: 10px 0;
}

.column:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0,0,0,0.15) !important;
}

.card {
  transition: all 0.3s ease;
  border-left: 4px solid #0d6efd;
}

.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.15) !important;
}

/* Drag and Drop Styles */
.sortable-ghost {
  opacity: 0.5;
  background: #f8f9fa;
}

.sortable-drag {
  opacity: 0.8;
  transform: rotate(5deg);
}

.form-control:focus {
  border-color: #0d6efd;
  box-shadow: 0 0 0 0.2rem rgba(13, 110, 253, 0.25);
}

@media (max-width: 768px) {
  .trello-board {
    padding: 0;
    width: 100vw;
    min-width: 100vw;
  }
  .board-content {
    width: 100vw;
    min-width: 100vw;
  }
  .d-flex.gap-3 {
    flex-direction: column !important;
    align-items: center;
  }
  .column {
    min-width: 100vw !important;
    max-width: 100vw !important;
    border-radius: 0 !important;
  }
}
</style> 