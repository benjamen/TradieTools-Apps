<!-- Business Management Dashboard - $9.99/month Bundle -->
<!-- Includes: Business Listing + Quotes + Invoices + Management -->

<template>
  <div class="dashboard">
    <!-- Header -->
    <div class="header">
      <div class="header-content">
        <img v-if="business.logo" :src="business.logo" class="logo">
        <div class="header-info">
          <h1>{{ business.business_name }}</h1>
          <p class="tag" :style="{ backgroundColor: business.brand_color }">{{ business.trade_type }}</p>
          <p class="subtitle">{{ business.service_area }}</p>
        </div>
        <div class="subscription-badge" :class="subscription.status">
          <span>⭐ {{ subscription.status === 'active' ? 'Professional' : 'Free' }}</span>
          <span>{{ subscription.status === 'active' ? '$9.99/month' : 'Upgrade' }}</span>
        </div>
      </div>
    </div>

    <!-- Quick Stats -->
    <div class="stats-grid">
      <div class="stat-card">
        <h3>Total Quotes</h3>
        <p class="stat-value">{{ quotes.length }}</p>
        <p class="stat-label">this month</p>
      </div>
      <div class="stat-card">
        <h3>Outstanding Invoices</h3>
        <p class="stat-value">${{ outstandingAmount.toFixed(2) }}</p>
        <p class="stat-label">{{ outstandingCount }} invoices</p>
      </div>
      <div class="stat-card">
        <h3>Total Revenue</h3>
        <p class="stat-value">${{ totalRevenue.toFixed(2) }}</p>
        <p class="stat-label">all time</p>
      </div>
      <div class="stat-card">
        <h3>Acceptance Rate</h3>
        <p class="stat-value">{{ acceptanceRate }}%</p>
        <p class="stat-label">{{ acceptedQuotes }}/{{ quotes.length }} quotes</p>
      </div>
    </div>

    <!-- Tabs -->
    <div class="tabs">
      <button
        v-for="tab in ['Overview', 'Quotes', 'Invoices', 'Profile']"
        :key="tab"
        :class="['tab', { active: activeTab === tab }]"
        @click="activeTab = tab">
        {{ tab }}
      </button>
    </div>

    <!-- Overview Tab -->
    <div v-if="activeTab === 'Overview'" class="tab-content">
      <div class="grid-2">
        <div class="card">
          <h3>Recent Quotes</h3>
          <div v-if="quotes.length === 0" class="empty-state">
            <p>No quotes yet. Create your first quote to get started!</p>
            <button class="btn btn-primary" @click="activeTab = 'Quotes'">Create Quote</button>
          </div>
          <div v-else class="quote-list">
            <div v-for="quote in quotes.slice(0, 5)" :key="quote.id" class="quote-item">
              <div>
                <p><strong>{{ quote.client_name }}</strong></p>
                <p class="small">${{ quote.total.toFixed(2) }}</p>
              </div>
              <span :class="['status', quote.status.toLowerCase()]">{{ quote.status }}</span>
            </div>
          </div>
        </div>

        <div class="card">
          <h3>Outstanding Invoices</h3>
          <div v-if="outstandingInvoices.length === 0" class="empty-state">
            <p>No outstanding invoices</p>
          </div>
          <div v-else class="invoice-list">
            <div v-for="invoice in outstandingInvoices.slice(0, 5)" :key="invoice.id" class="invoice-item">
              <div>
                <p><strong>{{ invoice.client_name }}</strong></p>
                <p class="small">${{ invoice.total_amount.toFixed(2) }}</p>
              </div>
              <span class="due">{{ daysOverdue(invoice.due_date) }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Quotes Tab -->
    <div v-if="activeTab === 'Quotes'" class="tab-content">
      <div class="card">
        <div class="card-header">
          <h3>Quotes</h3>
          <button class="btn btn-small" @click="showNewQuote = true">+ New Quote</button>
        </div>

        <table class="data-table">
          <thead>
            <tr>
              <th>Client</th>
              <th>Amount</th>
              <th>Status</th>
              <th>Date</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="quote in quotes" :key="quote.id">
              <td><strong>{{ quote.client_name }}</strong></td>
              <td>${{ quote.total.toFixed(2) }}</td>
              <td><span :class="['badge', quote.status.toLowerCase()]">{{ quote.status }}</span></td>
              <td>{{ formatDate(quote.created_at) }}</td>
              <td>
                <button class="action-btn" @click="viewQuote(quote)">View</button>
                <button class="action-btn" @click="createInvoiceFromQuote(quote)">Invoice</button>
                <button class="action-btn delete" @click="deleteQuote(quote.id)">Delete</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Invoices Tab -->
    <div v-if="activeTab === 'Invoices'" class="tab-content">
      <div class="card">
        <div class="card-header">
          <h3>Invoices</h3>
          <button class="btn btn-small" @click="showNewInvoice = true">+ New Invoice</button>
        </div>

        <table class="data-table">
          <thead>
            <tr>
              <th>Invoice #</th>
              <th>Client</th>
              <th>Amount</th>
              <th>Status</th>
              <th>Due Date</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="invoice in invoices" :key="invoice.id">
              <td><strong>{{ invoice.invoice_number }}</strong></td>
              <td>{{ invoice.client_name }}</td>
              <td>${{ invoice.total_amount.toFixed(2) }}</td>
              <td><span :class="['badge', invoice.status.toLowerCase()]">{{ invoice.status }}</span></td>
              <td>{{ formatDate(invoice.due_date) }}</td>
              <td>
                <button class="action-btn" @click="viewInvoice(invoice)">View</button>
                <button class="action-btn" @click="downloadInvoice(invoice)">PDF</button>
                <button class="action-btn" @click="sendInvoiceEmail(invoice)">Email</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Profile Tab -->
    <div v-if="activeTab === 'Profile'" class="tab-content">
      <div class="card">
        <h3>Business Profile</h3>
        <form @submit.prevent="saveProfile" class="profile-form">
          <div class="form-group">
            <label>Business Name *</label>
            <input v-model="business.business_name" required>
          </div>

          <div class="form-row">
            <div class="form-group">
              <label>Trade Type *</label>
              <select v-model="business.trade_type" required>
                <option value="Electrician">Electrician</option>
                <option value="Plumber">Plumber</option>
                <option value="Builder">Builder</option>
                <option value="Carpenter">Carpenter</option>
                <option value="Painter">Painter</option>
                <option value="HVAC">HVAC</option>
              </select>
            </div>
            <div class="form-group">
              <label>Service Area</label>
              <input v-model="business.service_area" placeholder="e.g., Auckland">
            </div>
          </div>

          <div class="form-row">
            <div class="form-group">
              <label>Phone</label>
              <input v-model="business.phone">
            </div>
            <div class="form-group">
              <label>Website</label>
              <input v-model="business.website" placeholder="https://...">
            </div>
          </div>

          <div class="form-group">
            <label>Description</label>
            <textarea v-model="business.description" placeholder="Tell clients about your business..."></textarea>
          </div>

          <div class="form-row">
            <div class="form-group">
              <label>Logo</label>
              <input type="file" @change="uploadLogo" accept="image/*">
            </div>
            <div class="form-group">
              <label>Brand Color</label>
              <input type="color" v-model="business.brand_color">
            </div>
          </div>

          <button type="submit" class="btn btn-primary">Save Profile</button>
        </form>
      </div>

      <div class="card">
        <h3>Subscription</h3>
        <div class="subscription-info">
          <p><strong>Plan:</strong> {{ subscription.status === 'active' ? 'Professional ($9.99/month)' : 'Free' }}</p>
          <p><strong>Status:</strong> <span :class="subscription.status">{{ subscription.status }}</span></p>
          <p v-if="subscription.renewalDate"><strong>Renews:</strong> {{ formatDate(subscription.renewalDate) }}</p>
          <button v-if="subscription.status === 'active'" class="btn btn-danger" @click="cancelSubscription">
            Cancel Subscription
          </button>
          <button v-else class="btn btn-primary" @click="upgradeSubscription">
            Upgrade to Professional
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// State
const activeTab = ref('Overview')
const business = ref({
  business_name: 'Your Business',
  trade_type: 'Electrician',
  service_area: 'Auckland',
  phone: '',
  website: '',
  description: '',
  logo: '',
  brand_color: '#0055a5'
})

const subscription = ref({
  status: 'free', // 'free' or 'active'
  renewalDate: null,
  subscriptionId: null
})

const quotes = ref([
  { id: 1, client_name: 'John Doe', total: 1500, status: 'Accepted', created_at: '2026-06-01' },
  { id: 2, client_name: 'Jane Smith', total: 2200, status: 'Sent', created_at: '2026-06-03' }
])

const invoices = ref([
  { id: 1, invoice_number: 'INV-001', client_name: 'John Doe', total_amount: 1500, status: 'Paid', due_date: '2026-06-15' },
  { id: 2, invoice_number: 'INV-002', client_name: 'Jane Smith', total_amount: 2200, status: 'Pending', due_date: '2026-06-10' }
])

const showNewQuote = ref(false)
const showNewInvoice = ref(false)

// Computed
const totalRevenue = computed(() => {
  return invoices.value
    .filter(inv => inv.status === 'Paid')
    .reduce((sum, inv) => sum + inv.total_amount, 0)
})

const outstandingAmount = computed(() => {
  return invoices.value
    .filter(inv => inv.status !== 'Paid' && inv.status !== 'Canceled')
    .reduce((sum, inv) => sum + inv.total_amount, 0)
})

const outstandingCount = computed(() => {
  return invoices.value.filter(inv => inv.status === 'Pending' || inv.status === 'Overdue').length
})

const outstandingInvoices = computed(() => {
  return invoices.value.filter(inv => inv.status !== 'Paid')
})

const acceptedQuotes = computed(() => {
  return quotes.value.filter(q => q.status === 'Accepted').length
})

const acceptanceRate = computed(() => {
  if (quotes.value.length === 0) return 0
  return Math.round((acceptedQuotes.value / quotes.value.length) * 100)
})

// Methods
const formatDate = (date) => {
  return new Date(date).toLocaleDateString('en-NZ')
}

const daysOverdue = (dueDate) => {
  const due = new Date(dueDate)
  const today = new Date()
  const days = Math.floor((today - due) / (1000 * 60 * 60 * 24))
  if (days > 0) return `${days} days overdue`
  if (days === 0) return 'Due today'
  return `Due in ${Math.abs(days)} days`
}

const viewQuote = (quote) => {
  alert(`Quote Details:\n${quote.client_name}\n$${quote.total.toFixed(2)}\nStatus: ${quote.status}`)
}

const viewInvoice = (invoice) => {
  alert(`Invoice Details:\n${invoice.invoice_number}\n${invoice.client_name}\n$${invoice.total_amount.toFixed(2)}`)
}

const downloadInvoice = (invoice) => {
  alert(`Downloading invoice ${invoice.invoice_number}...`)
}

const sendInvoiceEmail = (invoice) => {
  alert(`Email sent to ${invoice.client_name}`)
}

const createInvoiceFromQuote = (quote) => {
  const newInvoice = {
    id: invoices.value.length + 1,
    invoice_number: `INV-${String(invoices.value.length + 1).padStart(3, '0')}`,
    client_name: quote.client_name,
    total_amount: quote.total,
    status: 'Draft',
    due_date: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000).toISOString().split('T')[0]
  }
  invoices.value.push(newInvoice)
  alert(`Invoice created from quote: ${newInvoice.invoice_number}`)
  activeTab.value = 'Invoices'
}

const deleteQuote = (id) => {
  if (confirm('Delete this quote?')) {
    quotes.value = quotes.value.filter(q => q.id !== id)
  }
}

const saveProfile = () => {
  alert('✅ Profile saved')
}

const uploadLogo = (e) => {
  const file = e.target.files[0]
  if (file) {
    const reader = new FileReader()
    reader.onload = (event) => {
      business.value.logo = event.target.result
    }
    reader.readAsDataURL(file)
  }
}

const upgradeSubscription = () => {
  alert('Redirecting to Stripe checkout...')
  // window.location.href = '/stripe/subscribe'
}

const cancelSubscription = () => {
  if (confirm('Cancel subscription and downgrade to free?')) {
    subscription.value.status = 'free'
    alert('Subscription canceled')
  }
}
</script>

<style scoped>
* { margin: 0; padding: 0; box-sizing: border-box; }

.dashboard { background: #f5f5f5; min-height: 100vh; padding: 2rem 1rem; }

.header { background: white; padding: 2rem; border-radius: 8px; margin-bottom: 2rem; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
.header-content { display: flex; align-items: center; gap: 2rem; }
.logo { max-width: 100px; height: auto; }
.header-info h1 { font-size: 2em; color: #333; margin-bottom: 0.5rem; }
.tag { display: inline-block; padding: 0.5rem 1rem; border-radius: 4px; color: white; font-weight: 600; }
.subtitle { color: #666; font-size: 0.9em; }
.subscription-badge { text-align: center; padding: 1rem; background: #e8f5e9; border-radius: 6px; border-left: 4px solid #4caf50; }
.subscription-badge.free { background: #fff3e0; border-left-color: #ff9800; }
.subscription-badge span { display: block; }

.stats-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 1rem; margin-bottom: 2rem; }
.stat-card { background: white; padding: 1.5rem; border-radius: 8px; text-align: center; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
.stat-card h3 { color: #666; font-size: 0.9em; margin-bottom: 0.5rem; }
.stat-value { font-size: 2em; font-weight: bold; color: #0055a5; margin: 0.5rem 0; }
.stat-label { color: #999; font-size: 0.85em; }

@media (max-width: 1024px) {
  .stats-grid { grid-template-columns: repeat(2, 1fr); }
}

.tabs { display: flex; gap: 1rem; margin-bottom: 2rem; border-bottom: 2px solid #ddd; background: white; padding: 1rem; border-radius: 8px; }
.tab { background: none; border: none; padding: 1rem; cursor: pointer; color: #666; font-weight: 600; border-bottom: 3px solid transparent; margin-bottom: -1rem; }
.tab.active { color: #0055a5; border-bottom-color: #0055a5; }

.tab-content { background: white; padding: 2rem; border-radius: 8px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }

.grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 2rem; }
@media (max-width: 768px) { .grid-2 { grid-template-columns: 1fr; } }

.card { background: white; padding: 1.5rem; border-radius: 8px; margin-bottom: 2rem; border: 1px solid #eee; }
.card h3 { color: #333; margin-bottom: 1rem; }

.card-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem; }

.empty-state { text-align: center; padding: 2rem; color: #999; }

.quote-list, .invoice-list { }
.quote-item, .invoice-item { display: flex; justify-content: space-between; padding: 1rem; border-bottom: 1px solid #eee; }
.quote-item:last-child { border-bottom: none; }
.small { color: #999; font-size: 0.9em; margin-top: 0.25rem; }
.status { padding: 0.25rem 0.75rem; border-radius: 3px; font-size: 0.85em; font-weight: 600; }
.status.accepted { background: #c8e6c9; color: #2e7d32; }
.status.sent { background: #bbdefb; color: #1565c0; }
.status.pending { background: #fff9c4; color: #f57f17; }
.due { color: #d32f2f; font-weight: 600; }

.data-table { width: 100%; border-collapse: collapse; }
.data-table th { background: #f5f5f5; padding: 1rem; text-align: left; font-weight: 600; color: #666; }
.data-table td { padding: 1rem; border-bottom: 1px solid #eee; }
.data-table tr:hover { background: #f9f9f9; }

.badge { padding: 0.25rem 0.75rem; border-radius: 3px; font-size: 0.85em; font-weight: 600; }
.badge.accepted { background: #c8e6c9; color: #2e7d32; }
.badge.paid { background: #c8e6c9; color: #2e7d32; }
.badge.pending { background: #fff9c4; color: #f57f17; }
.badge.overdue { background: #ffcdd2; color: #c62828; }

.action-btn { background: none; border: none; color: #0055a5; cursor: pointer; font-weight: 600; padding: 0.25rem 0.5rem; }
.action-btn.delete { color: #d32f2f; }

.profile-form { }
.form-group { margin-bottom: 1rem; }
.form-group label { display: block; font-weight: 600; margin-bottom: 0.5rem; color: #333; }
.form-group input, .form-group select, .form-group textarea { width: 100%; padding: 0.75rem; border: 1px solid #ddd; border-radius: 4px; font-family: inherit; font-size: 1em; }
.form-row { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }

.subscription-info { background: #f9f9f9; padding: 1.5rem; border-radius: 6px; }
.subscription-info p { margin: 0.75rem 0; }
.subscription-info .active { color: #4caf50; font-weight: 600; }
.subscription-info .free { color: #ff9800; font-weight: 600; }

.btn { padding: 0.75rem 1.5rem; background: #0055a5; color: white; border: none; border-radius: 6px; cursor: pointer; font-weight: 600; }
.btn-small { padding: 0.5rem 1rem; font-size: 0.9em; }
.btn-primary { background: #0055a5; }
.btn-danger { background: #d32f2f; }
.btn:hover { opacity: 0.9; }
</style>
