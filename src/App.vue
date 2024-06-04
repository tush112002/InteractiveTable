<template>
  <div class="table-container">
    <h2>Student Information</h2>
    <div class="table-wrapper">
      <table class="student-table">
        <thead>
          <tr>
            <th>
              ID
              <button @click="sortBy('id')" class="sort-button">
                <Icon icon="fa6-solid:sort" class="sort-icon" />
              </button>
            </th>
            <th>
              Name
              <button @click="sortBy('name')" class="sort-button">
                <Icon icon="fa6-solid:sort" class="sort-icon" />
              </button>
            </th>
            <th>
              Age
              <button @click="sortBy('age')" class="sort-button">
                <Icon icon="fa6-solid:sort" class="sort-icon" />
              </button>
            </th>
            <th>
              Grade
            </th>
            <th>Action</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(student, index) in sortedStudents" :key="student.id">
            <td>
              <div class="icon-wrapper">
                {{ student.id }}
              </div>
            </td>
            <td>
              <div class="icon-wrapper">
                {{ student.name }}
              </div>
            </td>
            <td>
              <div class="icon-wrapper">
                {{ student.age }}
              </div>
            </td>
            <td>
              <div class="icon-wrapper">
                {{ student.grade }}
              </div>
            </td>
            <td>
              <button @click="deleteStudent(student.id, index)" class="delete-button">
                <Icon icon="icon-park:delete" class="delete-icon" />
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { Icon } from '@iconify/vue';
import axios from 'axios';

const students = ref([]);
const sortByKey = ref('');
const sortDirection = ref('asc');

const sortedStudents = computed(() => {
  if (!sortByKey.value) return students.value;

  return students.value.slice().sort((a, b) => {
    let modifier = 1;
    if (sortDirection.value === 'desc') modifier = -1;
    if (sortByKey.value === 'name') {
      return modifier * a[sortByKey.value].localeCompare(b[sortByKey.value]);
    } else {
      return modifier * (a[sortByKey.value] - b[sortByKey.value]);
    }
  });
});

const sortBy = (key) => {
  if (sortByKey.value === key) {
    sortDirection.value = sortDirection.value === 'asc' ? 'desc' : 'asc';
  } else {
    sortDirection.value = 'asc';
  }
  sortByKey.value = key;
};

const deleteStudent = async (id, index) => {
  try {
    if (confirm("Are you sure you want to delete this student?")) {
      const response = await axios.delete("http://localhost:3000/users/"+id);

      if (response.status !== 200) {
        throw new Error('Failed to delete student');
      }
      students.value.splice(index, 1);
    }
  } catch (error) {
    console.error('Error deleting student:', error);
  }
};

onMounted(async () => {
  try {
    const response = await axios.get('http://localhost:3000/users');
    students.value = response.data;
  } catch (error) {
    console.error('Error fetching data:', error);
  }
});
</script>

<style scoped>
.table-container {
  justify-content: center;
  margin-left: 50%;
  width: max-content;
}

.table-wrapper {
  overflow-x: auto;
}

.student-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
  border-radius: 5px;
  border: 1px solid #ccc;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
}

.student-table th,
.student-table td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

.student-table th {
  background-color: #f4f4f4;
}

.student-table tbody tr:nth-child(even) {
  background-color: #f9f9f9;
}

.student-table tbody tr:hover {
  background-color: #f0f0f0;
}

.sort-button,
.delete-button {
  padding: 0;
  background: none;
  border: none;
}

.sort-icon,
.delete-icon {
  vertical-align: middle;
  margin-left: 100%;
  margin-bottom: 40%;
}

.icon-wrapper {
  display: flex;
  align-items: center;
}

.sort-icon svg,
.delete-icon svg {
  width: 1em;
  height: 1em;
}

.delete-icon svg path {
  fill: #f44336;
}
</style>
