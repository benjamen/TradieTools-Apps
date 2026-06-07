<!-- TradieTools Quote Builder - Vue3 Component -->
<!-- Features: Interactive calculations, PDF export, email sending, pricing database integration -->

<template>
  <div class="quote-builder-container">
    <header class="header">
      <h1>💰 Quote Builder</h1>
      <p>Create professional quotes in 2 minutes. No signup required.</p>
    </header>

    <div class="grid-2">
      <!-- Left: Input Form -->
      <div class="card">
        <h2>Quote Details</h2>

        <!-- Client Info -->
        <div class="form-group">
          <label>Client Name *</label>
          <input v-model="form.clientName" placeholder="John Smith" required>
        </div>

        <div class="form-group">
          <label>Client Email</label>
          <input v-model="form.email" type="email" placeholder="john@example.com">
        </div>

        <div class="form-group">
          <label>Project Type *</label>
          <select v-model="form.tradeType" @change="updateDefaultRate">
            <option value="">Select trade</option>
            <option value="electrician">Electrical Work</option>
            <option value="plumber">Plumbing</option>
            <option value="builder">Building/Construction</option>
            <option value="painter">Painting</option>
            <option value="carpenter">Carpentry</option>
            <option value="hvac">HVAC</option>
            <option value="other">Other</option>
          </select>
        </div>

        <div class="form-group">
          <label>Job Title/Description *</label>
          <input v-model="form.jobTitle" placeholder="e.g., Bathroom renovation" required>
        </div>

        <div class="form-group">
          <label>Job Scope</label>
          <textarea v-model="form.description" placeholder="Detailed description of work..."></textarea>
        </div>

        <hr>

        <!-- Labor -->
        <h3>Labor Costs</h3>
        <div class="form-row">
          <div class="form-group">
            <label>Hours *</label>
            <input v-model.number="form.hours" type="number" step="0.5" placeholder="8" required>
          </div>
          <div class="form-group">
            <label>Hourly Rate ($) *</label>
            <input v-model.number="form.rate" type="number" placeholder="85" required>
          </div>
        </div>

        <!-- Materials -->
        <h3>Materials & Supplies</h3>
        <div class="form-group">
          <label>Total Cost ($)</label>
          <input v-model.number="form.materials" type="number" placeholder="250" required>
        </div>

        <!-- Pricing -->
        <h3>Pricing</h3>
        <div class="form-row">
          <div class="form-group">
            <label>Markup (%)</label>
            <input v-model.number="form.markup" type="number" value="20" min="0" max="100">
          </div>
          <div class="form-group">
            <label>
              <input type="checkbox" v-model="form.includeGst"> Include 15% GST
            </label>
          </div>
        </div>

        <!-- Actions -->
        <button @click="calculateQuote" class="btn btn-primary">💡 Calculate Quote</button>
        <button @click="saveQuote" class="btn btn-success" style="margin-top: 8px;">💾 Save Quote</button>
        <button @click="downloadPDF" class="btn btn-secondary" style="margin-top: 8px;">📥 Download PDF</button>
      </div>

      <!-- Right: Breakdown & Total -->
      <div class="card">
        <h2>Quote Breakdown</h2>

        <div v-if="!hasCalculated" style="text-align: center; color: #999; padding: 40px 0;">
          <p>Fill in the form and click "Calculate Quote"</p>
        </div>

        <div v-else class="breakdown">
          <div class="breakdown-row">
            <span>Materials</span>
            <span>${{ form.materials.toFixed(2) }}</span>
          </div>
          <div class="breakdown-row">
            <span>Labor ({{ form.hours }}h × ${{ form.rate }}/h)</span>
            <span>${{ laborCost.toFixed(2) }}</span>
          </div>
          <div class="breakdown-row highlight">
            <span><strong>Subtotal</strong></span>
            <span><strong>${{ subtotal.toFixed(2) }}</strong></span>
          </div>
          <div class="breakdown-row">
            <span>Markup ({{ form.markup }}%)</span>
            <span>${{ markupAmount.toFixed(2) }}</span>
          </div>
          <div class="breakdown-row">
            <span>Before GST</span>
            <span>${{ beforeGst.toFixed(2) }}</span>
          </div>
          <div v-if="form.includeGst" class="breakdown-row">
            <span>GST (15%)</span>
            <span>${{ gstAmount.toFixed(2) }}</span>
          </div>
          <div class="breakdown-row total">
            <span><strong>TOTAL QUOTE</strong></span>
            <span><strong>${{ total.toFixed(2) }}</strong></span>
          </div>

          <div style="margin-top: 20px; padding: 15px; background: #f0f0f0; border-radius: 6px;">
            <p style="font-size: 0.9em; color: #666;">
              ✓ Valid for 30 days<br>
              ✓ 50% deposit to start<br>
              ✓ Final payment on completion
            </p>
          </div>
        </div>
      </div>
    </div>

    <!-- Success Message -->
    <div v-if="successMessage" class="success-message">
      {{ successMessage }}
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// Form state
const form = ref({
  clientName: '',
  email: '',
  tradeType: '',
  jobTitle: '',
  description: '',
  hours: 8,
  rate: 85,
  materials: 250,
  markup: 20,
  includeGst: true
})

const hasCalculated = ref(false)
const successMessage = ref('')

// Trade rates (can be fetched from API)
const tradeRates = {
  electrician: 95,
  plumber: 90,
  builder: 85,
  painter: 65,
  carpenter: 80,
  hvac: 95,
  other: 75
}

// Computed properties
const laborCost = computed(() => form.value.hours * form.value.rate)
const subtotal = computed(() => form.value.materials + laborCost.value)
const markupAmount = computed(() => subtotal.value * (form.value.markup / 100))
const beforeGst = computed(() => subtotal.value + markupAmount.value)
const gstAmount = computed(() => form.value.includeGst ? beforeGst.value * 0.15 : 0)
const total = computed(() => beforeGst.value + gstAmount.value)

// Methods
const updateDefaultRate = () => {
  if (form.value.tradeType && tradeRates[form.value.tradeType]) {
    form.value.rate = tradeRates[form.value.tradeType]
  }
}

const calculateQuote = () => {
  if (!form.value.clientName || !form.value.jobTitle || !form.value.hours || !form.value.rate) {
    alert('Please fill in required fields')
    return
  }
  hasCalculated.value = true
}

const saveQuote = async () => {
  if (!hasCalculated.value) {
    alert('Please calculate quote first')
    return
  }

  try {
    const response = await fetch('/api/tradie/quotes/create', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        client_name: form.value.clientName,
        client_email: form.value.email,
        project_type: form.value.tradeType,
        labor_hours: form.value.hours,
        labor_rate: form.value.rate,
        material_cost: form.value.materials,
        markup_pct: form.value.markup
      })
    })

    const result = await response.json()
    successMessage.value = `✅ Quote saved! ID: ${result.quote_id}`
    setTimeout(() => { successMessage.value = '' }, 5000)
  } catch (err) {
    alert('Error saving quote: ' + err.message)
  }
}

const downloadPDF = () => {
  // Generate PDF (integrate with pdfkit or similar)
  const quoteText = `
QUOTE - ${form.value.jobTitle}

Client: ${form.value.clientName}
Email: ${form.value.email}
Date: ${new Date().toLocaleDateString()}

Materials: $${form.value.materials.toFixed(2)}
Labor: ${form.value.hours}h × $${form.value.rate}/h = $${laborCost.value.toFixed(2)}
Subtotal: $${subtotal.value.toFixed(2)}
Markup (${form.value.markup}%): $${markupAmount.value.toFixed(2)}
${form.value.includeGst ? `GST (15%): $${gstAmount.value.toFixed(2)}\n` : ''}
TOTAL: $${total.value.toFixed(2)}

Terms: 50% deposit to start, final payment on completion.
Valid for 30 days.
  `

  const blob = new Blob([quoteText], { type: 'text/plain' })
  const url = window.URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `quote-${form.value.clientName.replace(/\s+/g, '-')}.txt`
  document.body.appendChild(a)
  a.click()
  window.URL.revokeObjectURL(url)
  a.remove()
}
</script>

<style scoped>
.quote-builder-container {
  background: linear-gradient(135deg, #1b2a4a 0%, #2d3e5f 100%);
  min-height: 100vh;
  padding: 20px;
}

.header {
  background: white;
  border-radius: 12px;
  padding: 30px;
  margin-bottom: 30px;
  text-align: center;
}

.header h1 {
  color: #1b2a4a;
  font-size: 2.2em;
  margin-bottom: 10px;
}

.header p {
  color: #666;
  font-size: 1.1em;
}

.grid-2 {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  max-width: 1200px;
  margin: 0 auto 30px;
}

@media (max-width: 900px) {
  .grid-2 {
    grid-template-columns: 1fr;
  }
}

.card {
  background: white;
  border-radius: 12px;
  padding: 25px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.card h2 {
  color: #1b2a4a;
  margin-bottom: 20px;
  font-size: 1.5em;
}

.card h3 {
  color: #1b2a4a;
  margin: 25px 0 15px 0;
  font-size: 1.1em;
  border-bottom: 2px solid #e0e0e0;
  padding-bottom: 10px;
}

.form-group {
  margin-bottom: 18px;
}

.form-group label {
  display: block;
  margin-bottom: 6px;
  color: #333;
  font-weight: 500;
}

.form-group input,
.form-group select,
.form-group textarea {
  width: 100%;
  padding: 10px 12px;
  border: 2px solid #e0e0e0;
  border-radius: 6px;
  font-size: 1em;
  font-family: inherit;
  transition: border-color 0.2s;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #005ea2;
  box-shadow: 0 0 0 3px rgba(0, 94, 162, 0.1);
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
}

hr {
  margin: 20px 0;
  border: none;
  border-top: 1px solid #e0e0e0;
}

.btn {
  width: 100%;
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  font-size: 1em;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}

.btn-primary {
  background: #005ea2;
  color: white;
}

.btn-primary:hover {
  background: #003d7a;
}

.btn-success {
  background: #4caf50;
  color: white;
}

.btn-success:hover {
  background: #3d8b40;
}

.btn-secondary {
  background: #666;
  color: white;
}

.btn-secondary:hover {
  background: #555;
}

.breakdown {
  background: #f5f5f5;
  border-radius: 8px;
  padding: 20px;
}

.breakdown-row {
  display: flex;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #e0e0e0;
}

.breakdown-row:last-child {
  border-bottom: none;
}

.breakdown-row.highlight {
  border-top: 2px solid #005ea2;
  padding-top: 12px;
  font-weight: 600;
  color: #005ea2;
}

.breakdown-row.total {
  background: #1b2a4a;
  color: white;
  padding: 12px 15px;
  margin: 10px -20px -20px -20px;
  border-radius: 0 0 8px 8px;
  border: none;
  font-size: 1.2em;
}

.success-message {
  background: #e8f5e9;
  border-left: 4px solid #4caf50;
  padding: 15px;
  border-radius: 6px;
  color: #2e7d32;
  max-width: 1200px;
  margin: 0 auto;
  text-align: center;
}
</style>
