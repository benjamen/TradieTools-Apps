<!-- Quote Builder with Stripe Payment Integration -->
<!-- Features: Branded quotes, freemium model (3 free/month), Stripe checkout -->

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

    <!-- Subscription Status Banner -->
    <div class="status-banner" :class="subscription.isPaid ? 'premium' : 'free'">
      <div v-if="subscription.isPaid" class="status-content">
        <span class="badge">⭐ PROFESSIONAL</span>
        <p>Unlimited quotes • Custom branding • Priority support</p>
        <a href="#" @click.prevent="openManageSubscription">Manage subscription</a>
      </div>
      <div v-else class="status-content">
        <span class="badge">FREE TIER</span>
        <p>{{ quotaRemaining }} quotes remaining this month</p>
        <button v-if="quotaRemaining > 0" class="upgrade-btn-small" @click="showPaymentModal = true">
          Upgrade for $9.99/month
        </button>
        <button v-else class="upgrade-btn-small" @click="showPaymentModal = true">
          Upgrade Now to Continue
        </button>
      </div>
    </div>

    <!-- Quote Form -->
    <div class="quote-form" :style="{ borderColor: brandColor }">
      <h2>Create Professional Quote</h2>

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
          :disabled="!canCreateQuote && !subscription.isPaid">
          {{ !canCreateQuote && !subscription.isPaid ? '❌ Quota Reached' : '👁️ Preview Quote' }}
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

    <!-- Payment Modal (Stripe Checkout) -->
    <div v-if="showPaymentModal" class="modal-overlay">
      <div class="modal" :style="{ borderColor: brandColor }">
        <button class="modal-close" @click="closePaymentModal">✕</button>

        <h2>Upgrade to Professional</h2>
        <p>Unlimited quotes + custom branding for just $9.99/month</p>

        <!-- Payment Form -->
        <form @submit.prevent="processPayment" class="payment-form">
          <!-- Email -->
          <div class="form-group">
            <label>Email Address *</label>
            <input v-model="payment.email" type="email" placeholder="you@example.com" required>
          </div>

          <!-- Name -->
          <div class="form-group">
            <label>Full Name *</label>
            <input v-model="payment.name" type="text" placeholder="Your Name" required>
          </div>

          <!-- Stripe Card Element -->
          <div class="form-group">
            <label>Card Details *</label>
            <div id="card-element" class="stripe-element"></div>
            <div v-if="payment.cardError" class="error-message">{{ payment.cardError }}</div>
          </div>

          <!-- Pricing Summary -->
          <div class="pricing-summary">
            <div class="price-row">
              <span>Professional Plan</span>
              <span>$9.99 NZD</span>
            </div>
            <div class="price-row">
              <span>Billing Cycle</span>
              <span>Monthly (Cancel anytime)</span>
            </div>
            <div class="price-row total">
              <span>Monthly Total</span>
              <span>$9.99 NZD</span>
            </div>
          </div>

          <!-- Submit Button -->
          <button
            type="submit"
            class="btn btn-upgrade"
            :disabled="payment.isProcessing"
            :style="{ backgroundColor: brandColor }">
            {{ payment.isProcessing ? '⏳ Processing...' : '✅ Subscribe Now' }}
          </button>

          <!-- Trust Badges -->
          <div class="trust-badges">
            <span>🔒 Secure payment via Stripe</span>
            <span>📋 Cancel anytime</span>
          </div>
        </form>

        <!-- Pricing Comparison -->
        <div class="pricing-comparison">
          <h3>What's Included</h3>
          <table>
            <tr>
              <th>Feature</th>
              <th>Free</th>
              <th>Professional</th>
            </tr>
            <tr>
              <td>Quotes per month</td>
              <td>3</td>
              <td>∞ Unlimited</td>
            </tr>
            <tr>
              <td>Custom branding</td>
              <td>❌</td>
              <td>✅</td>
            </tr>
            <tr>
              <td>Email delivery</td>
              <td>❌</td>
              <td>✅</td>
            </tr>
            <tr>
              <td>Quote history</td>
              <td>❌</td>
              <td>✅</td>
            </tr>
            <tr>
              <td>Priority support</td>
              <td>❌</td>
              <td>✅</td>
            </tr>
          </table>
        </div>
      </div>
    </div>

    <!-- Subscription Management Modal -->
    <div v-if="showSubscriptionManagement" class="modal-overlay">
      <div class="modal" :style="{ borderColor: brandColor }">
        <button class="modal-close" @click="showSubscriptionManagement = false">✕</button>

        <h2>Manage Subscription</h2>

        <div class="subscription-details">
          <p><strong>Plan:</strong> Professional ($9.99 NZD/month)</p>
          <p><strong>Status:</strong> <span class="badge active">Active</span></p>
          <p><strong>Renews:</strong> {{ subscription.renewalDate }}</p>
          <p><strong>Quotes Used This Month:</strong> Unlimited</p>
        </div>

        <div class="actions">
          <button class="btn btn-secondary" @click="openStripePortal">
            💳 Update Payment Method
          </button>
          <button class="btn btn-danger" @click="cancelSubscription">
            ❌ Cancel Subscription
          </button>
        </div>

        <p class="fine-print">
          No refunds for partial months. Cancel anytime and downgrade to free tier.
        </p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

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

const userEmail = ref('')
const subscription = ref({
  isPaid: false,
  renewalDate: null,
  subscriptionId: null
})

const payment = ref({
  email: '',
  name: '',
  cardError: '',
  isProcessing: false
})

const showPreview = ref(false)
const showPaymentModal = ref(false)
const showSubscriptionManagement = ref(false)
const quotaRemaining = ref(3)

// Stripe
let stripe = null
let cardElement = null

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
const canCreateQuote = computed(() => subscription.value.isPaid || quotaRemaining.value > 0)

// Lifecycle
onMounted(async () => {
  // Initialize Stripe
  initializeStripe()

  // Check user email from localStorage
  userEmail.value = localStorage.getItem('userEmail') || ''

  // Load subscription status
  if (userEmail.value) {
    await loadSubscriptionStatus()
    await checkQuotaRemaining()
  }
})

// Methods
const initializeStripe = () => {
  if (typeof Stripe === 'undefined') {
    console.error('Stripe.js not loaded')
    return
  }

  stripe = Stripe('YOUR_STRIPE_PUBLISHABLE_KEY')
  const elements = stripe.elements()
  cardElement = elements.create('card')

  // Mount card element after modal opens
  const observer = new MutationObserver(() => {
    const cardDiv = document.getElementById('card-element')
    if (cardDiv && !cardDiv.hasChildNodes()) {
      cardElement.mount('#card-element')
    }
  })

  observer.observe(document.body, { childList: true, subtree: true })
}

const uploadLogo = (e) => {
  const file = e.target.files[0]
  if (file) {
    const reader = new FileReader()
    reader.onload = (event) => { logo.value = event.target.result }
    reader.readAsDataURL(file)
  }
}

const generateQuote = async () => {
  if (!form.value.clientName || !form.value.jobTitle) {
    alert('Fill in client name and project title')
    return
  }

  // Check quota
  if (!subscription.value.isPaid && quotaRemaining.value <= 0) {
    alert('You\'ve reached your free tier limit (3 quotes/month). Upgrade to continue.')
    showPaymentModal.value = true
    return
  }

  showPreview.value = true

  // Decrement quota for free tier
  if (!subscription.value.isPaid) {
    quotaRemaining.value--
    localStorage.setItem('quotasUsed', String(3 - quotaRemaining.value))
  }
}

const downloadPDF = () => {
  const quoteText = `
QUOTE - ${form.value.jobTitle}

From: ${company.value.name}
${company.value.phone} | ${company.value.email}

To: ${form.value.clientName}
Project: ${form.value.jobTitle}
Date: ${new Date().toLocaleDateString()}

BREAKDOWN:
Materials: $${form.value.materials.toFixed(2)}
Labor: ${form.value.hours}h × $${form.value.rate}/h = $${laborCost.value.toFixed(2)}
Subtotal: $${subtotal.value.toFixed(2)}
Markup (${form.value.markup}%): $${markupAmount.value.toFixed(2)}
GST (15%): $${gst.value.toFixed(2)}

TOTAL: $${total.value.toFixed(2)}

TERMS:
Deposit: $${(total.value * form.value.deposit / 100).toFixed(2)} (${form.value.deposit}%)
Valid for: ${form.value.validDays} days
Payment: ${form.value.deposit}% deposit to start, balance on completion
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

const emailQuote = async () => {
  if (!form.value.email) {
    alert('Enter client email')
    return
  }

  try {
    const response = await fetch('/api/tradie/quotes/email', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        client_email: form.value.email,
        client_name: form.value.clientName,
        quote_total: total.value
      })
    })

    if (response.ok) {
      alert('✅ Quote sent to ' + form.value.email)
    } else {
      alert('Failed to send email')
    }
  } catch (err) {
    alert('Error: ' + err.message)
  }
}

const processPayment = async (e) => {
  e.preventDefault()

  if (!stripe || !cardElement) {
    alert('Payment system not initialized')
    return
  }

  payment.value.isProcessing = true
  payment.value.cardError = ''

  try {
    // Create payment method
    const { paymentMethod, error } = await stripe.createPaymentMethod({
      type: 'card',
      card: cardElement,
      billing_details: {
        email: payment.value.email,
        name: payment.value.name
      }
    })

    if (error) {
      payment.value.cardError = error.message
      payment.value.isProcessing = false
      return
    }

    // Create subscription via backend
    const response = await fetch('/api/stripe/subscribe', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        email: payment.value.email,
        name: payment.value.name,
        payment_method_id: paymentMethod.id
      })
    })

    if (response.ok) {
      const result = await response.json()

      // Store subscription info
      localStorage.setItem('userEmail', payment.value.email)
      localStorage.setItem('subscriptionId', result.subscription_id)
      localStorage.setItem('subscriptionStatus', result.status)

      // Update UI
      userEmail.value = payment.value.email
      subscription.value.isPaid = true
      subscription.value.subscriptionId = result.subscription_id
      subscription.value.renewalDate = result.current_period_end
      quotaRemaining.value = null

      alert('✅ Welcome to Professional! You now have unlimited quotes.')
      closePaymentModal()
    } else {
      const error = await response.json()
      payment.value.cardError = error.detail || 'Subscription failed'
    }
  } catch (err) {
    payment.value.cardError = 'Error: ' + err.message
  } finally {
    payment.value.isProcessing = false
  }
}

const loadSubscriptionStatus = async () => {
  try {
    const response = await fetch(`/api/stripe/subscription/${userEmail.value}`)
    if (response.ok) {
      const data = await response.json()
      subscription.value.isPaid = data.is_active
      subscription.value.renewalDate = data.current_period_end
      subscription.value.subscriptionId = data.subscription_id
    }
  } catch (err) {
    console.error('Failed to load subscription:', err)
  }
}

const checkQuotaRemaining = async () => {
  try {
    const response = await fetch(`/api/stripe/check-quota?user_email=${userEmail.value}`)
    if (response.ok) {
      const data = await response.json()
      quotaRemaining.value = data.quotes_remaining || 0
    }
  } catch (err) {
    console.error('Failed to check quota:', err)
  }
}

const closePaymentModal = () => {
  showPaymentModal.value = false
  payment.value = { email: '', name: '', cardError: '', isProcessing: false }
}

const openManageSubscription = () => {
  showSubscriptionManagement.value = true
}

const openStripePortal = () => {
  // In production, call /api/stripe/customer-portal
  alert('Redirecting to billing portal...')
}

const cancelSubscription = async () => {
  if (!confirm('Cancel Professional subscription and downgrade to free tier?')) {
    return
  }

  try {
    const response = await fetch(`/api/stripe/cancel-subscription/${subscription.value.subscriptionId}`, {
      method: 'POST'
    })

    if (response.ok) {
      // Update UI
      subscription.value.isPaid = false
      quotaRemaining.value = 3
      localStorage.removeItem('subscriptionId')
      localStorage.removeItem('subscriptionStatus')

      alert('✅ Subscription canceled. You\'ve been downgraded to free tier.')
      showSubscriptionManagement.value = false
    }
  } catch (err) {
    alert('Error: ' + err.message)
  }
}
</script>

<style scoped>
* { margin: 0; padding: 0; box-sizing: border-box; }

.quote-builder { background: #f5f5f5; min-height: 100vh; padding: 2rem 1rem; }

.branding-section { background: white; padding: 2rem; border-radius: 8px; margin-bottom: 2rem; box-shadow: 0 1px 3px rgba(0,0,0,0.1); max-width: 1200px; margin: 0 auto 2rem; }
.logo-upload { margin-bottom: 1.5rem; }
.logo-preview { margin-top: 1rem; }
.logo-preview img { max-width: 200px; height: auto; }
.brand-inputs { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; margin: 1rem 0; }
.company-name { grid-column: 1 / -1; font-size: 1.3em; font-weight: bold; }
.brand-inputs input { padding: 0.75rem; border: 1px solid #ddd; border-radius: 4px; }
.brand-colors { margin-top: 1rem; }
.brand-colors input[type="color"] { width: 60px; height: 40px; cursor: pointer; }

.status-banner { background: white; padding: 1.5rem; border-radius: 8px; margin-bottom: 2rem; max-width: 1200px; margin: 0 auto 2rem; border-left: 4px solid; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
.status-banner.free { border-left-color: #ff9800; background: #fff3e0; }
.status-banner.premium { border-left-color: #4caf50; background: #e8f5e9; }
.status-content { display: flex; align-items: center; justify-content: space-between; gap: 2rem; }
.badge { padding: 0.5rem 1rem; border-radius: 4px; font-weight: bold; font-size: 0.9em; }
.status-banner.free .badge { background: #ff9800; color: white; }
.status-banner.premium .badge { background: #4caf50; color: white; }
.status-banner p { color: #666; margin: 0; }
.status-banner a { color: #0055a5; text-decoration: none; font-weight: 600; }
.upgrade-btn-small { background: #ff9800; color: white; border: none; padding: 0.5rem 1.5rem; border-radius: 4px; cursor: pointer; font-weight: 600; }
.upgrade-btn-small:hover { background: #f57c00; }

.quote-form { background: white; padding: 2rem; border-radius: 8px; border-top: 4px solid; max-width: 1200px; margin: 0 auto 2rem; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
.quote-form h2 { color: #333; margin-bottom: 2rem; }
.form-group { margin-bottom: 1.5rem; }
.form-group label { display: block; font-weight: 600; margin-bottom: 0.5rem; color: #333; }
.form-group input, .form-group textarea { width: 100%; padding: 0.75rem; border: 1px solid #ddd; border-radius: 4px; font-size: 1em; font-family: inherit; }
.form-group input:focus, .form-group textarea:focus { outline: none; border-color: #0055a5; box-shadow: 0 0 0 3px rgba(0,85,165,0.1); }
.form-row { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
@media (max-width: 600px) { .form-row { grid-template-columns: 1fr; } }

.costs-section, .terms-section { background: #f9f9f9; padding: 1.5rem; border-radius: 6px; margin: 1.5rem 0; }
.costs-section h3, .terms-section h3 { color: #0055a5; margin-bottom: 1rem; }

.quote-preview { background: white; border: 1px solid #ddd; border-top: 4px solid; padding: 2rem; margin: 2rem 0; border-radius: 6px; }
.quote-header { background: #0055a5; color: white; padding: 2rem; border-radius: 4px; margin-bottom: 2rem; display: flex; gap: 2rem; align-items: center; }
.logo { flex-shrink: 0; }
.logo img { max-width: 100px; height: auto; }
.company-info h1 { margin: 0; font-size: 1.8em; }
.company-info p { margin: 0.5rem 0; font-size: 0.9em; }

.quote-details { margin-bottom: 1.5rem; }
.quote-title { font-size: 1.5em; font-weight: bold; color: #333; margin-bottom: 1rem; }
.quote-details p { margin: 0.5rem 0; color: #666; }

.quote-items { width: 100%; border-collapse: collapse; margin: 1.5rem 0; }
.quote-items th { background: #f0f0f0; padding: 0.75rem; text-align: left; font-weight: bold; border-bottom: 2px solid #ddd; }
.quote-items td { padding: 0.75rem; border-bottom: 1px solid #ddd; }
.quote-items td:last-child { text-align: right; }
.subtotal-row td { font-weight: bold; border-top: 2px solid #333; }
.total-row { background: #0055a5; color: white; font-weight: bold; font-size: 1.1em; }
.total-row td { border: none; }

.quote-terms { background: #f9f9f9; padding: 1rem; border-radius: 6px; margin-top: 1.5rem; }
.quote-terms p { margin: 0.5rem 0; color: #666; }

.actions { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 1rem; margin-top: 2rem; }
@media (max-width: 600px) { .actions { grid-template-columns: 1fr; } }

.btn { padding: 0.75rem 1.5rem; border: 2px solid; border-radius: 6px; font-weight: 600; cursor: pointer; font-size: 1em; }
.btn-primary { background: white; color: white; border: none; }
.btn-primary:hover:not(:disabled) { opacity: 0.9; }
.btn-primary:disabled { opacity: 0.5; cursor: not-allowed; }
.btn-secondary { background: white; color: #333; border: 2px solid; }
.btn-secondary:hover { background: #f5f5f5; }

.modal-overlay { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.7); display: flex; align-items: center; justify-content: center; z-index: 1000; padding: 1rem; }
.modal { background: white; padding: 2rem; border-radius: 8px; border-top: 4px solid; max-width: 500px; width: 100%; max-height: 90vh; overflow-y: auto; position: relative; }
.modal-close { position: absolute; top: 1rem; right: 1rem; background: none; border: none; font-size: 1.5em; cursor: pointer; color: #666; }
.modal h2 { margin-bottom: 0.5rem; color: #333; }
.modal > p:nth-child(3) { color: #666; margin-bottom: 1.5rem; }

.payment-form { margin: 2rem 0; }
.payment-form .form-group { margin-bottom: 1.5rem; }
.stripe-element { border: 1px solid #ddd; padding: 1rem; border-radius: 4px; background: #f9f9f9; }
.error-message { color: #d32f2f; font-size: 0.9em; margin-top: 0.5rem; }

.pricing-summary { background: #f5f5f5; padding: 1rem; border-radius: 6px; margin: 1.5rem 0; }
.price-row { display: flex; justify-content: space-between; padding: 0.75rem 0; border-bottom: 1px solid #ddd; }
.price-row.total { font-weight: bold; color: #0055a5; border: none; border-top: 2px solid #ddd; padding-top: 1rem; }

.btn-upgrade { width: 100%; margin-top: 1rem; }
.btn-danger { background: #d32f2f; color: white; border: none; }
.btn-danger:hover { background: #b71c1c; }

.trust-badges { display: flex; justify-content: center; gap: 2rem; margin-top: 1rem; font-size: 0.9em; color: #666; }

.pricing-comparison { margin-top: 2rem; border-top: 1px solid #ddd; padding-top: 2rem; }
.pricing-comparison h3 { color: #333; margin-bottom: 1rem; }
.pricing-comparison table { width: 100%; border-collapse: collapse; }
.pricing-comparison th { background: #f0f0f0; padding: 0.75rem; text-align: left; font-weight: bold; }
.pricing-comparison td { padding: 0.75rem; border-bottom: 1px solid #ddd; }

.subscription-details { background: #e8f5e9; padding: 1.5rem; border-radius: 6px; margin: 1.5rem 0; }
.subscription-details p { margin: 0.5rem 0; }
.badge.active { background: #4caf50; color: white; padding: 0.25rem 0.75rem; border-radius: 3px; font-size: 0.9em; }

.fine-print { color: #999; font-size: 0.85em; margin-top: 1.5rem; }
</style>
