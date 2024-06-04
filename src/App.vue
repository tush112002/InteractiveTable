<template>
  <div class="table-container">
    <h2 class="header">Student Information</h2>
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
              <button @click="updateStudentForm(student)" class="update-button">
                <Icon icon="icon-park:edit" class="update-icon" />
              </button>
            </td>
          </tr>
          <tr v-if="showAddForm">
            <td>
              <input type="text" v-model="newStudent.id" placeholder="ID" />
            </td>
            <td>
              <input type="text" v-model="newStudent.name" placeholder="Name" />
            </td>
            <td>
              <input type="number" v-model="newStudent.age" placeholder="Age" />
            </td>
            <td>
              <input type="text" v-model="newStudent.grade" placeholder="Grade" />
            </td>
            <td>
              <button @click="addStudent" class="save-button">Save</button>
              <button @click="cancelAdd" class="cancel-button">Cancel</button>
            </td>
          </tr>
          <tr v-if="showUpdateForm">
            <td>
              {{ updatedStudent.id }}
            </td>
            <td>
              <input type="text" v-model="updatedStudent.name" placeholder="Name" />
            </td>
            <td>
              <input type="number" v-model="updatedStudent.age" placeholder="Age" />
            </td>
            <td>
              <input type="text" v-model="updatedStudent.grade" placeholder="Grade" />
            </td>
            <td>
              <button @click="saveUpdatedStudent" class="save-button">Save</button>
              <button @click="cancelUpdate" class="cancel-button">Cancel</button>
            </td>
          </tr>
        </tbody>
      </table>
      <button @click="showAddForm = true" v-if="!showAddForm" class="add-button">Add Student</button>
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
const showAddForm = ref(false);
const showUpdateForm = ref(false);
const newStudent = ref({
  id: '',
  name: '',
  age: '',
  grade: ''
});
const updatedStudent = ref({
  id: '',
  name: '',
  age: '',
  grade: ''
});

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
      const response = await axios.delete(`http://localhost:3000/users/${id}`);

      if (response.status !== 200) {
        throw new Error('Failed to delete student');
      }
      students.value.splice(index, 1);
    }
  } catch (error) {
    console.error('Error deleting student:', error);
  }
};

const addStudent = async () => {
  try
  {
    const response = await axios.post('http://localhost:3000/users', newStudent.value);
    if (response.status !== 201) {
      throw new Error('Failed to add student');
    }
    students.value.push(response.data);
    newStudent.value = { id: '', name: '', age: '', grade: '' };
    showAddForm.value = false;
  } catch (error) {
    console.error('Error adding student:', error);
  }
};

const cancelAdd = () => {
  showAddForm.value = false;
  newStudent.value = { id: '', name: '', age: '', grade: '' };
};

const updateStudentForm = (student) => {
  updatedStudent.value = { ...student };
  showUpdateForm.value = true;
};

const saveUpdatedStudent = async () => {
  try {
    const { id, ...updatedData } = updatedStudent.value;
    const response = await axios.put(`http://localhost:3000/users/${id}`, updatedData);
    if (response.status !== 200) {
      throw new Error('Failed to update student');
    }

    const index = students.value.findIndex(student => student.id === updatedStudent.value.id);
    if (index !== -1) {
      students.value.splice(index, 1, response.data);
    } else {
      console.error('Student not found in the array');
    }
    showUpdateForm.value = false;
  } catch (error) {
    console.error('Error updating student:', error);
  }
};

const cancelUpdate = () => {
  showUpdateForm.value = false;
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
  flex-direction: column;
  align-items: center;
  margin-left: 300px;
  width: 900px;
}

.header {
  margin-left: 30%;
  margin-bottom: 10px;
}

.table-wrapper {
  overflow-x: auto;
  width: 100%;
}

.student-table {
  width: 80%;
  border-collapse: collapse;
  margin-top: 20px;
  border: 2px solid #ddd; 
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
}

.student-table th,
.student-table td {
  padding: 12px 15px;
  text-align: center;
  border: 1px solid #ddd; 
}

.student-table th:last-child,
.student-table td:last-child {
  border-right: none;
}

.student-table tbody tr {
  border-bottom: 1px solid #ddd;
}

.student-table tbody tr:last-child {
  border-bottom: none;
}

.student-table thead tr th:first-child {
  border-top-left-radius: 8px;
}

.student-table thead tr th:last-child {
  border-top-right-radius: 8px; 
}

.student-table tbody tr:last-child td:first-child {
  border-bottom-left-radius: 8px; 
}

.student-table tbody tr:last-child td:last-child {
  border-bottom-right-radius: 8px; 
}

.icon-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
}

.sort-button,
.delete-button,
.save-button,
.cancel-button,
.update-button {
  padding: 0;
  background: none;
  border: none;
  cursor: pointer;
}

.sort-icon,
.delete-icon,
.update-icon {
  vertical-align: middle;
  margin-left: 10px;
}

.sort-icon svg,
.delete-icon svg,
.update-icon svg {
  width: 1em;
  height: 1em;
}

.delete-icon svg path {
  fill: #f44336;
}

.add-button {
  margin-top: 20px;
  padding: 10px 20px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.add-button:hover {
  background-color: #45a049;
}

input[type="text"],
input[type="number"] {
  width: 90%;
  padding: 8px;
  margin: 2px 0;
  box-sizing: border-box;
}

.save-button,
.cancel-button {
  background-color: #4caf50;
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
  border: none;
  display: inline-block;
}

.cancel-button {
  background-color: #f44336;
  margin-top: 5px;
}

.save-button:hover {
  background-color: #45a049;
}

.cancel-button:hover {
  background-color: #e53935;
}
</style>
