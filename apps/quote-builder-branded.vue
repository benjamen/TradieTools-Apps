<!-- Quote Builder with Branding + Freemium Model -->
<template>
  <div class="quote-builder">
    <!-- Branding Section -->
    <div class="branding-section">
      <div class="logo-upload">
        <label>Your Company Logo</label>
        <input type="file" @change="uploadLogo" accept="image/*">
        <div v-if="logo" class="logo-preview">
          <img :src="logo" alt="Logo">
        </div>
      </div>

      <div class="brand-inputs">
        <input v-model="company.name" placeholder="Company Name" class="company-name">
        <input v-model="company.abn" placeholder="ABN/Tax ID">
        <input v-model="company.phone" placeholder="Phone">
        <input v-model="company.email" placeholder="Email">
        <input v-model="company.website" placeholder="Website">
      </div>

      <div class="brand-colors">
        <label>Brand Color</label>
        <input type="color" v-model="brandColor">
      </div>
    </div>

    <!-- Quote Form -->
    <div class="quote-form" :style="{ borderColor: brandColor }">
      <h2>Create Professional Quote</h2>

      <div class="quota-info" v-if="!isPaid">
        <p>🎁 Free: {{ quotesUsed }}/3 quotes used this month</p>
        <div v-if="quotesUsed >= 3" class="upgrade-prompt">
          <p>Upgrade to create unlimited quotes</p>
          <button @click="showUpgrade = true" class="btn-upgrade">Upgrade Now</button>
        </div>
      </div>

      <div class="form-group">
        <label>Client Name *</label>
        <input v-model="form.clientName" placeholder="John Smith">
      </div>

      <div class="form-row">
        <div class="form-group">
          <label>Client Email</label>
          <input v-model="form.email" type="email">
        </div>
        <div class="form-group">
          <label>Phone</label>
          <input v-model="form.phone" type="tel">
        </div>
      </div>

      <div class="form-group">
        <label>Project Title *</label>
        <input v-model="form.jobTitle" placeholder="Bathroom Renovation">
      </div>

      <div class="form-group">
        <label>Description</label>
        <textarea v-model="form.description" placeholder="Detailed description..."></textarea>
      </div>

      <!-- Costs -->
      <div class="costs-section">
        <h3>Costs</h3>
        <div class="form-row">
          <div class="form-group">
            <label>Materials ($)</label>
            <input v-model.number="form.materials" type="number" placeholder="0">
          </div>
          <div class="form-group">
            <label>Hours</label>
            <input v-model.number="form.hours" type="number" step="0.5" placeholder="0">
          </div>
        </div>
        <div class="form-row">
          <div class="form-group">
            <label>Hourly Rate ($)</label>
            <input v-model.number="form.rate" type="number" placeholder="85">
          </div>
          <div class="form-group">
            <label>Markup (%)</label>
            <input v-model.number="form.markup" type="number" placeholder="20">
          </div>
        </div>
      </div>

      <!-- Terms -->
      <div class="terms-section">
        <h3>Terms</h3>
        <div class="form-row">
          <div class="form-group">
            <label>Deposit (%)</label>
            <input v-model.number="form.deposit" type="number" value="50">
          </div>
          <div class="form-group">
            <label>Valid for (days)</label>
            <input v-model.number="form.validDays" type="number" value="30">
          </div>
        </div>
      </div>

      <!-- Quote Preview -->
      <div v-if="showPreview" class="quote-preview" :style="{ borderTopColor: brandColor }">
        <div class="quote-header" :style="{ backgroundColor: brandColor }">
          <div v-if="logo" class="logo"><img :src="logo" alt="Logo"></div>
          <div class="company-info">
            <h1>{{ company.name || 'Your Company' }}</h1>
            <p>{{ company.phone }} | {{ company.email }}</p>
            <p>{{ company.website }}</p>
          </div>
        </div>

        <div class="quote-details">
          <div class="quote-title">QUOTE</div>
          <p><strong>To:</strong> {{ form.clientName }}</p>
          <p><strong>Project:</strong> {{ form.jobTitle }}</p>
          <p><strong>Date:</strong> {{ new Date().toLocaleDateString() }}</p>
        </div>

        <table class="quote-items">
          <tr>
            <th>Description</th>
            <th>Qty</th>
            <th>Amount</th>
          </tr>
          <tr>
            <td>Materials & Supplies</td>
            <td>1</td>
            <td>${{ form.materials.toFixed(2) }}</td>
          </tr>
          <tr>
            <td>Labor ({{ form.hours }}h @ ${{ form.rate }}/h)</td>
            <td>1</td>
            <td>${{ laborCost.toFixed(2) }}</td>
          </tr>
          <tr class="subtotal-row">
            <td colspan="2"><strong>Subtotal</strong></td>
            <td><strong>${{ subtotal.toFixed(2) }}</strong></td>
          </tr>
          <tr>
            <td colspan="2">Markup ({{ form.markup }}%)</td>
            <td>${{ markupAmount.toFixed(2) }}</td>
          </tr>
          <tr>
            <td colspan="2">GST (15%)</td>
            <td>${{ gst.toFixed(2) }}</td>
          </tr>
          <tr class="total-row">
            <td colspan="2"><strong>TOTAL</strong></td>
            <td><strong>${{ total.toFixed(2) }}</strong></td>
          </tr>
        </table>

        <div class="quote-terms">
          <p><strong>Deposit Required:</strong> ${{ (total * form.deposit / 100).toFixed(2) }} ({{ form.deposit }}%)</p>
          <p><strong>Due Date:</strong> {{ dueDate }}</p>
          <p><strong>Terms:</strong> {{ form.deposit }}% deposit to start, balance on completion</p>
        </div>
      </div>

      <!-- Actions -->
      <div class="actions">
        <button
          @click="generateQuote"
          class="btn btn-primary"
          :style="{ backgroundColor: brandColor }"
          :disabled="quotesUsed >= 3 && !isPaid">
          👁️ Preview Quote
        </button>
        <button
          v-if="showPreview"
          @click="downloadPDF"
          class="btn btn-secondary"
          :style="{ borderColor: brandColor }">
          📥 Download PDF
        </button>
        <button
          v-if="showPreview"
          @click="emailQuote"
          class="btn btn-secondary"
          :style="{ borderColor: brandColor }">
          📧 Email Quote
        </button>
      </div>
    </div>

    <!-- Upgrade Modal -->
    <div v-if="showUpgrade" class="modal-overlay">
      <div class="modal" :style="{ borderColor: brandColor }">
        <h2>Upgrade to Professional</h2>
        <p>Unlock unlimited quotes + branded templates</p>

        <div class="pricing">
          <div class="plan">
            <h3>Free</h3>
            <p class="price">$0/month</p>
            <ul>
              <li>3 quotes/month</li>
              <li>Basic templates</li>
            </ul>
          </div>
          <div class="plan featured">
            <h3>Professional</h3>
            <p class="price">$9.99/month</p>
            <ul>
              <li>Unlimited quotes</li>
              <li>Branded templates</li>
              <li>Logo & colors</li>
              <li>Email delivery</li>
              <li>Quote history</li>
            </ul>
            <button class="btn btn-upgrade" @click="upgradeToPaid">Get Professional</button>
          </div>
        </div>

        <button @click="showUpgrade = false" class="btn-close">Close</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// State
const company = ref({ name: '', abn: '', phone: '', email: '', website: '' })
const logo = ref('')
const brandColor = ref('#0055a5')
const form = ref({
  clientName: '',
  email: '',
  phone: '',
  jobTitle: '',
  description: '',
  materials: 0,
  hours: 0,
  rate: 85,
  markup: 20,
  deposit: 50,
  validDays: 30
})
const showPreview = ref(false)
const showUpgrade = ref(false)
const isPaid = ref(false)
const quotesUsed = ref(0)

// Computed
const laborCost = computed(() => form.value.hours * form.value.rate)
const subtotal = computed(() => form.value.materials + laborCost.value)
const markupAmount = computed(() => subtotal.value * (form.value.markup / 100))
const beforeGst = computed(() => subtotal.value + markupAmount.value)
const gst = computed(() => beforeGst.value * 0.15)
const total = computed(() => beforeGst.value + gst.value)
const dueDate = computed(() => {
  const d = new Date()
  d.setDate(d.getDate() + form.value.validDays)
  return d.toLocaleDateString()
})

// Methods
const uploadLogo = (e) => {
  const file = e.target.files[0]
  if (file) {
    const reader = new FileReader()
    reader.onload = (event) => { logo.value = event.target.result }
    reader.readAsDataURL(file)
  }
}

const generateQuote = () => {
  if (!form.value.clientName || !form.value.jobTitle) {
    alert('Fill in client name and project title')
    return
  }
  if (quotesUsed.value >= 3 && !isPaid.value) {
    showUpgrade.value = true
    return
  }
  showPreview.value = true
  quotesUsed.value++
}

const downloadPDF = () => {
  alert('PDF download - integrate pdfkit/html2pdf')
}

const emailQuote = async () => {
  const resp = await fetch('/api/tradie/quotes/email', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      client_email: form.value.email,
      client_name: form.value.clientName,
      total: total.value
    })
  })
  alert('Quote sent!')
}

const upgradeToPaid = () => {
  // Stripe integration
  window.location.href = '/stripe/subscribe'
}
</script>

<style scoped>
.quote-builder { max-width: 1000px; margin: 0 auto; padding: 2rem; }
.branding-section { background: white; padding: 2rem; border-radius: 8px; margin-bottom: 2rem; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
.logo-upload { margin-bottom: 1.5rem; }
.logo-preview { margin-top: 1rem; }
.logo-preview img { max-width: 200px; height: auto; }
.brand-inputs { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; margin: 1rem 0; }
.company-name { grid-column: 1 / -1; font-size: 1.3em; font-weight: bold; }
.brand-colors { margin-top: 1rem; }
.brand-colors input[type="color"] { width: 60px; height: 40px; }

.quote-form { background: white; padding: 2rem; border-radius: 8px; border-top: 4px solid; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
.quota-info { background: #f0f0f0; padding: 1rem; border-radius: 6px; margin-bottom: 1.5rem; }
.upgrade-prompt { background: #fff3cd; padding: 1rem; border-radius: 6px; margin-top: 0.5rem; }

.form-group { margin-bottom: 1rem; }
.form-group label { display: block; font-weight: 600; margin-bottom: 0.5rem; }
.form-group input, .form-group textarea { width: 100%; padding: 0.75rem; border: 1px solid #ddd; border-radius: 4px; }

.form-row { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }

.costs-section, .terms-section { background: #f9f9f9; padding: 1.5rem; border-radius: 6px; margin: 1.5rem 0; }
.costs-section h3, .terms-section h3 { color: #333; margin-bottom: 1rem; }

.quote-preview { background: white; border: 1px solid #ddd; border-top: 4px solid; padding: 2rem; margin: 2rem 0; border-radius: 6px; }
.quote-header { background: #0055a5; color: white; padding: 2rem; border-radius: 4px; margin-bottom: 2rem; display: flex; gap: 2rem; align-items: center; }
.logo { flex-shrink: 0; }
.logo img { max-width: 100px; }
.company-info h1 { margin: 0; font-size: 1.8em; }
.company-info p { margin: 0.5rem 0; font-size: 0.9em; }

.quote-details { margin-bottom: 1.5rem; }
.quote-title { font-size: 1.5em; font-weight: bold; color: #333; }
.quote-items { width: 100%; border-collapse: collapse; margin: 1.5rem 0; }
.quote-items th { background: #f0f0f0; padding: 0.75rem; text-align: left; font-weight: bold; }
.quote-items td { padding: 0.75rem; border-bottom: 1px solid #ddd; }
.quote-items td:last-child { text-align: right; }
.subtotal-row td { font-weight: bold; border-top: 2px solid #333; }
.total-row { background: #0055a5; color: white; font-weight: bold; font-size: 1.1em; }
.total-row td { border: none; }

.quote-terms { background: #f9f9f9; padding: 1rem; border-radius: 6px; margin-top: 1.5rem; }
.quote-terms p { margin: 0.5rem 0; }

.actions { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 1rem; margin-top: 2rem; }
.btn { padding: 0.75rem 1.5rem; border: 2px solid; border-radius: 6px; font-weight: 600; cursor: pointer; }
.btn-primary { background: white; color: white; border: none; }
.btn-primary:disabled { opacity: 0.5; cursor: not-allowed; }
.btn-secondary { background: white; color: #333; }

.modal-overlay { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.5); display: flex; align-items: center; justify-content: center; z-index: 1000; }
.modal { background: white; padding: 2rem; border-radius: 8px; border-top: 4px solid; max-width: 600px; }
.modal h2 { margin-bottom: 0.5rem; }
.modal p { color: #666; margin-bottom: 1.5rem; }

.pricing { display: grid; grid-template-columns: 1fr 1fr; gap: 1.5rem; margin: 2rem 0; }
.plan { padding: 1.5rem; border: 1px solid #ddd; border-radius: 6px; text-align: center; }
.plan.featured { border: 2px solid #0055a5; background: #f0f7ff; }
.plan h3 { margin: 0; }
.plan .price { font-size: 1.8em; font-weight: bold; color: #0055a5; margin: 0.5rem 0; }
.plan ul { list-style: none; text-align: left; margin: 1rem 0; }
.plan li { padding: 0.5rem 0; }

.btn-upgrade { width: 100%; background: #0055a5; color: white; border: none; padding: 0.75rem; border-radius: 4px; font-weight: bold; cursor: pointer; }
.btn-close { width: 100%; margin-top: 1rem; padding: 0.75rem; background: #f0f0f0; border: 1px solid #ddd; border-radius: 4px; cursor: pointer; }
</style>
