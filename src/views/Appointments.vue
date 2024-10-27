<script setup lang="ts">
import { ref } from 'vue'
import { useQuery, useMutation } from '@vue/apollo-composable'
import { GET_APPOINTMENTS } from '../graphql/queries'
import { CREATE_APPOINTMENT } from '../graphql/mutations'
import BaseModal from '../components/modals/BaseModal.vue'
import AppointmentForm from '../components/forms/AppointmentForm.vue'

type AppointmentStatus = 'Geplant' | 'In Behandlung' | 'Abgeschlossen' | 'Storniert'

interface Appointment {
  id: string
  patientName: string
  doctorName: string
  date: string
  time: string
  status: AppointmentStatus
}

const isAddAppointmentModalOpen = ref(false)

const { result: appointmentsResult, loading, error } = useQuery<{
  appointments: Appointment[]
}>(GET_APPOINTMENTS)

const { mutate: createAppointment } = useMutation(CREATE_APPOINTMENT)

const statusColors = {
  'Geplant': 'bg-blue-100 text-blue-800',
  'In Behandlung': 'bg-yellow-100 text-yellow-800',
  'Abgeschlossen': 'bg-green-100 text-green-800',
  'Storniert': 'bg-red-100 text-red-800'
} as const

const handleAddAppointment = async (appointmentData: Partial<Appointment>) => {
  try {
    await createAppointment({
      variables: {
        input: appointmentData
      },
      refetchQueries: [{ query: GET_APPOINTMENTS }]
    })
    isAddAppointmentModalOpen.value = false
  } catch (err) {
    console.error('Error creating appointment:', err)
  }
}
</script>

<template>
  <div class="p-6">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-2xl font-bold text-gray-900">Termine</h1>
      <button
        @click="isAddAppointmentModalOpen = true"
        class="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700"
      >
        Termin vereinbaren
      </button>
    </div>

    <BaseModal
      :is-open="isAddAppointmentModalOpen"
      title="Termin vereinbaren"
      @close="isAddAppointmentModalOpen = false"
    >
      <AppointmentForm
        @submit="handleAddAppointment"
        @cancel="isAddAppointmentModalOpen = false"
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
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Patient</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Arzt</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Datum</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Zeit</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
          </tr>
        </thead>
        <tbody class="bg-white divide-y divide-gray-200">
          <tr v-for="appointment in appointmentsResult?.appointments" :key="appointment.id">
            <td class="px-6 py-4 whitespace-nowrap">{{ appointment.patientName }}</td>
            <td class="px-6 py-4 whitespace-nowrap">{{ appointment.doctorName }}</td>
            <td class="px-6 py-4 whitespace-nowrap">{{ appointment.date }}</td>
            <td class="px-6 py-4 whitespace-nowrap">{{ appointment.time }}</td>
            <td class="px-6 py-4 whitespace-nowrap">
              <span :class="[statusColors[appointment.status as AppointmentStatus], 'px-3 py-1 rounded-full text-sm']">
                {{ appointment.status }}
              </span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>