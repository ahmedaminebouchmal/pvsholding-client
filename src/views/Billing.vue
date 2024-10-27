<script setup lang="ts">
import { ref } from 'vue'
import { useQuery, useMutation } from '@vue/apollo-composable'
import { GET_BILLS } from '../graphql/queries'
import { CREATE_BILL } from '../graphql/mutations'
import BaseModal from '../components/modals/BaseModal.vue'
import BillingForm from '../components/forms/BillingForm.vue'

type BillStatus = 'Bezahlt' | 'Ausstehend' | 'Überfällig'

interface BillItem {
  description: string
  amount: number
}

interface Bill {
  id: string
  patientName: string
  date: string
  items: BillItem[]
  totalAmount: number
  status: BillStatus
}

const isAddBillModalOpen = ref(false)

const { result: billsResult, loading, error } = useQuery<{
  bills: Bill[]
}>(GET_BILLS)

const { mutate: createBill } = useMutation(CREATE_BILL)

const statusColors = {
  'Bezahlt': 'bg-green-100 text-green-800',
  'Ausstehend': 'bg-yellow-100 text-yellow-800',
  'Überfällig': 'bg-red-100 text-red-800'
} as const

const formatCurrency = (amount: number) => {
  return new Intl.NumberFormat('de-DE', {
    style: 'currency',
    currency: 'EUR'
  }).format(amount)
}

const handleAddBill = async (billData: Partial<Bill>) => {
  try {
    await createBill({
      variables: {
        input: billData
      },
      refetchQueries: [{ query: GET_BILLS }]
    })
    isAddBillModalOpen.value = false
  } catch (err) {
    console.error('Error creating bill:', err)
  }
}
</script>

<template>
  <div class="p-6">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-2xl font-bold text-gray-900">Abrechnung</h1>
      <button
        @click="isAddBillModalOpen = true"
        class="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700"
      >
        Neue Rechnung erstellen
      </button>
    </div>

    <BaseModal
      :is-open="isAddBillModalOpen"
      title="Neue Rechnung erstellen"
      @close="isAddBillModalOpen = false"
    >
      <BillingForm
        @submit="handleAddBill"
        @cancel="isAddBillModalOpen = false"
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
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Datum</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Leistungen</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Betrag</th>
            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
          </tr>
        </thead>
        <tbody class="bg-white divide-y divide-gray-200">
          <tr v-for="bill in billsResult?.bills" :key="bill.id">
            <td class="px-6 py-4 whitespace-nowrap">{{ bill.patientName }}</td>
            <td class="px-6 py-4 whitespace-nowrap">{{ bill.date }}</td>
            <td class="px-6 py-4">{{ bill.items.map((item: BillItem) => item.description).join(', ') }}</td>
            <td class="px-6 py-4 whitespace-nowrap">{{ formatCurrency(bill.totalAmount) }}</td>
            <td class="px-6 py-4 whitespace-nowrap">
              <span :class="[statusColors[bill.status as BillStatus], 'px-3 py-1 rounded-full text-sm']">
                {{ bill.status }}
              </span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>