<script setup lang="ts">
import { ref } from 'vue'
import { useQuery, useMutation } from '@vue/apollo-composable'
import { GET_STAFF } from '../graphql/queries'
import { CREATE_STAFF } from '../graphql/mutations'
import BaseModal from '../components/modals/BaseModal.vue'
import StaffForm from '../components/forms/StaffForm.vue'

type StaffStatus = 'Verfügbar' | 'In Behandlung' | 'Im Dienst' | 'Nicht im Dienst'

interface StaffMember {
  id: string
  name: string
  role: string
  department: string
  contact: string
  status: StaffStatus
}

const isAddStaffModalOpen = ref(false)

const { result: staffResult, loading, error } = useQuery<{
  staff: StaffMember[]
}>(GET_STAFF)

const { mutate: createStaff } = useMutation(CREATE_STAFF)

const statusColors = {
  'Verfügbar': 'bg-green-100 text-green-800',
  'In Behandlung': 'bg-yellow-100 text-yellow-800',
  'Im Dienst': 'bg-blue-100 text-blue-800',
  'Nicht im Dienst': 'bg-gray-100 text-gray-800'
} as const

const handleAddStaff = async (staffData: Partial<StaffMember>) => {
  try {
    await createStaff({
      variables: {
        input: staffData
      },
      refetchQueries: [{ query: GET_STAFF }]
    })
    isAddStaffModalOpen.value = false
  } catch (err) {
    console.error('Error creating staff member:', err)
  }
}
</script>

<template>
  <div class="p-6">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-2xl font-bold text-gray-900">Personalverwaltung</h1>
      <button
        @click="isAddStaffModalOpen = true"
        class="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700"
      >
        Personal hinzufügen
      </button>
    </div>

    <BaseModal
      :is-open="isAddStaffModalOpen"
      title="Personal hinzufügen"
      @close="isAddStaffModalOpen = false"
    >
      <StaffForm
        @submit="handleAddStaff"
        @cancel="isAddStaffModalOpen = false"
      />
    </BaseModal>

    <div v-if="loading" class="text-center py-4">
      Laden...
    </div>
    <div v-else-if="error" class="text-red-600 text-center py-4">
      Ein Fehler ist aufgetreten
    </div>
    <div v-else class="bg-white shadow-md rounded-lg overflow-hidden">
      <table class="min-w-full divide-y divide-gray-200">
        <thead class="bg-gray-50">
          <tr>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Name</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Position</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Abteilung</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Kontakt</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
          </tr>
        </thead>
        <tbody class="bg-white divide-y divide-gray-200">
          <tr v-for="member in staffResult?.staff" :key="member.id">
            <td class="px-6 py-4 whitespace-nowrap">{{ member.name }}</td>
            <td class="px-6 py-4 whitespace-nowrap">{{ member.role }}</td>
            <td class="px-6 py-4 whitespace-nowrap">{{ member.department }}</td>
            <td class="px-6 py-4 whitespace-nowrap">{{ member.contact }}</td>
            <td class="px-6 py-4 whitespace-nowrap">
              <span :class="[statusColors[member.status as StaffStatus], 'px-3 py-1 rounded-full text-sm']">
                {{ member.status }}
              </span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>