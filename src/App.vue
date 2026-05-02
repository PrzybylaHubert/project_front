<script setup>
import { computed, onMounted, ref, watch } from 'vue'

const STORAGE_KEY = 'planer-nauki-plany'
const LAST_REMOVED_KEY = 'planer-nauki-ostatnio-usunięty'

const activeView = ref('dashboard')
const plans = ref([])
const lastRemoved = ref(null)
const status = ref(null)
let statusTimer = null
const form = ref({
  subject: '',
  topic: '',
  date: '',
  duration: 45,
  priority: 'Średni',
})
const errors = ref({})

const views = [
  { id: 'dashboard', label: 'Dashboard' },
  { id: 'add', label: 'Dodaj plan' },
  { id: 'summary', label: 'Podsumowanie' },
]

const today = computed(() => new Date().toISOString().slice(0, 10))
const sortedPlans = computed(() =>
  [...plans.value].sort((a, b) => a.date.localeCompare(b.date)),
)
const upcomingPlans = computed(() =>
  sortedPlans.value.filter((plan) => plan.date >= today.value),
)
const totalMinutes = computed(() =>
  plans.value.reduce((sum, plan) => sum + Number(plan.duration), 0),
)
const plannedSubjects = computed(() => new Set(plans.value.map((plan) => plan.subject)).size)
const nextPlan = computed(() => upcomingPlans.value[0] || null)

const samplePlans = computed(() => [
  {
    subject: 'Matematyka',
    topic: 'Granice funkcji',
    date: today.value,
    duration: 60,
    priority: 'Wysoki',
  },
  {
    subject: 'Programowanie',
    topic: 'Komponenty Vue',
    date: today.value,
    duration: 45,
    priority: 'Średni',
  },
])

onMounted(() => {
  const savedPlans = localStorage.getItem(STORAGE_KEY)
  const savedRemoved = localStorage.getItem(LAST_REMOVED_KEY)

  if (savedPlans) {
    plans.value = JSON.parse(savedPlans)
  }

  if (savedRemoved) {
    lastRemoved.value = JSON.parse(savedRemoved)
  }
})

watch(
  plans,
  (value) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(value))
  },
  { deep: true },
)

watch(
  lastRemoved,
  (value) => {
    if (value) {
      localStorage.setItem(LAST_REMOVED_KEY, JSON.stringify(value))
      return
    }

    localStorage.removeItem(LAST_REMOVED_KEY)
  },
  { deep: true },
)

function setView(viewId) {
  activeView.value = viewId
  errors.value = {}
  clearStatus()
}

function validateForm() {
  const nextErrors = {}

  if (!form.value.subject.trim()) {
    nextErrors.subject = 'Wpisz nazwę przedmiotu.'
  }

  if (!form.value.topic.trim()) {
    nextErrors.topic = 'Wpisz temat nauki.'
  }

  if (!form.value.date) {
    nextErrors.date = 'Wybierz termin nauki.'
  } else if (form.value.date < today.value) {
    nextErrors.date = 'Termin nie może być z przeszłości.'
  }

  if (!form.value.duration || Number(form.value.duration) < 15) {
    nextErrors.duration = 'Czas nauki musi mieć co najmniej 15 minut.'
  }

  errors.value = nextErrors
  return Object.keys(nextErrors).length === 0
}

function submitPlan() {
  if (!validateForm()) {
    showStatus({
      type: 'error',
      text: 'Nie zapisano planu. Popraw pola oznaczone komunikatem.',
    })
    return
  }

  const newPlan = {
    id: crypto.randomUUID(),
    subject: form.value.subject.trim(),
    topic: form.value.topic.trim(),
    date: form.value.date,
    duration: Number(form.value.duration),
    priority: form.value.priority,
    createdAt: new Date().toISOString(),
  }

  plans.value.push(newPlan)
  lastRemoved.value = null
  resetForm()
  showStatus({
    type: 'success',
    text: 'Plan zapisany. Możesz go sprawdzić w podsumowaniu.',
  })
  activeView.value = 'summary'
}

function resetForm() {
  form.value = {
    subject: '',
    topic: '',
    date: '',
    duration: 45,
    priority: 'Średni',
  }
  errors.value = {}
}

function cancelForm() {
  resetForm()
  showStatus({
    type: 'info',
    text: 'Wprowadzanie anulowane. Formularz został wyczyszczony.',
  })
  activeView.value = 'dashboard'
}

function removePlan(planId) {
  const plan = plans.value.find((item) => item.id === planId)

  if (!plan) {
    return
  }

  plans.value = plans.value.filter((item) => item.id !== planId)
  lastRemoved.value = plan
  showStatus({
    type: 'warning',
    text: 'Plan usunięty. Możesz cofnąć tę operację.',
  })
}

function undoRemove() {
  if (!lastRemoved.value) {
    return
  }

  plans.value.push(lastRemoved.value)
  lastRemoved.value = null
  showStatus({
    type: 'success',
    text: 'Cofnięto usunięcie planu.',
  })
}

function addSamplePlans() {
  const samples = samplePlans.value.map((plan) => ({
    ...plan,
    id: crypto.randomUUID(),
    createdAt: new Date().toISOString(),
  }))

  plans.value.push(...samples)
  showStatus({
    type: 'success',
    text: 'Dodano przykładowe plany nauki.',
  })
}

function clearAllPlans() {
  plans.value = []
  lastRemoved.value = null
  showStatus({
    type: 'warning',
    text: 'Wyczyszczono wszystkie plany.',
  })
}

function showStatus(nextStatus) {
  clearTimeout(statusTimer)
  status.value = nextStatus
  statusTimer = setTimeout(() => {
    status.value = null
    statusTimer = null
  }, 5000)
}

function clearStatus() {
  clearTimeout(statusTimer)
  status.value = null
  statusTimer = null
}

function formatDate(date) {
  return new Intl.DateTimeFormat('pl-PL', {
    day: '2-digit',
    month: 'long',
    year: 'numeric',
  }).format(new Date(`${date}T12:00:00`))
}
</script>

<template>
  <div class="app-shell">
    <header class="topbar">
      <div>
        <p class="eyebrow">Projekt UI zgodny z heurystykami Nielsena</p>
        <h1>Planer Nauki</h1>
      </div>

      <nav class="nav" aria-label="Główne widoki aplikacji">
        <button
          v-for="view in views"
          :key="view.id"
          type="button"
          :class="['nav-button', { active: activeView === view.id }]"
          @click="setView(view.id)"
        >
          {{ view.label }}
        </button>
      </nav>
    </header>

    <main>
      <section v-if="status" :class="['system-status', status.type]" aria-live="polite">
        <span>{{ status.text }}</span>
        <button
          v-if="lastRemoved"
          type="button"
          class="small-button"
          @click="undoRemove"
        >
          Cofnij
        </button>
      </section>

      <section v-if="activeView === 'dashboard'" class="view-grid">
        <div class="intro-panel">
          <p class="eyebrow">Dashboard</p>
          <h2>Dzisiejszy plan jest pod kontrolą</h2>
          <p>
            Sprawdź najbliższą naukę, dodaj nowy plan albo przejdź do pełnego
            podsumowania. Najważniejsze akcje są widoczne od razu.
          </p>
          <div class="actions">
            <button type="button" class="primary-button" @click="setView('add')">
              Dodaj plan
            </button>
            <button type="button" class="secondary-button" @click="setView('summary')">
              Zobacz podsumowanie
            </button>
          </div>
        </div>

        <div class="stats-grid" aria-label="Statystyki planu nauki">
          <article class="stat-tile">
            <span>Liczba planów</span>
            <strong>{{ plans.length }}</strong>
          </article>
          <article class="stat-tile">
            <span>Przedmioty</span>
            <strong>{{ plannedSubjects }}</strong>
          </article>
          <article class="stat-tile">
            <span>Łączny czas</span>
            <strong>{{ totalMinutes }} min</strong>
          </article>
        </div>

        <section class="content-panel">
          <div class="section-title">
            <h2>Najbliższy termin</h2>
            <span v-if="nextPlan" class="badge">{{ nextPlan.priority }}</span>
          </div>

          <div v-if="nextPlan" class="next-plan">
            <strong>{{ nextPlan.subject }}</strong>
            <span>{{ nextPlan.topic }}</span>
            <small>{{ formatDate(nextPlan.date) }} · {{ nextPlan.duration }} min</small>
          </div>

          <p v-else class="empty-state">
            Nie masz jeszcze zaplanowanej nauki. Dodaj pierwszy plan albo użyj
            danych przykładowych.
          </p>

          <button
            v-if="plans.length === 0"
            type="button"
            class="secondary-button"
            @click="addSamplePlans"
          >
            Dodaj przykładowe plany
          </button>
        </section>
      </section>

      <section v-if="activeView === 'add'" class="form-layout">
        <div class="content-panel">
          <p class="eyebrow">Formularz</p>
          <h2>Dodaj plan nauki</h2>
          <p class="helper-text">
            Pola wymagane są oznaczone gwiazdką. Komunikaty pod polami pokazują,
            jak naprawić błąd.
          </p>

          <form class="study-form" novalidate @submit.prevent="submitPlan">
            <label>
              <span>Przedmiot *</span>
              <input
                v-model="form.subject"
                type="text"
                placeholder="np. Matematyka"
                :aria-invalid="Boolean(errors.subject)"
              />
              <small v-if="errors.subject" class="field-error">{{ errors.subject }}</small>
            </label>

            <label>
              <span>Temat nauki *</span>
              <input
                v-model="form.topic"
                type="text"
                placeholder="np. Pochodne funkcji"
                :aria-invalid="Boolean(errors.topic)"
              />
              <small v-if="errors.topic" class="field-error">{{ errors.topic }}</small>
            </label>

            <div class="form-row">
              <label>
                <span>Termin *</span>
                <input
                  v-model="form.date"
                  type="date"
                  :min="today"
                  :aria-invalid="Boolean(errors.date)"
                />
                <small v-if="errors.date" class="field-error">{{ errors.date }}</small>
              </label>

              <label>
                <span>Czas nauki *</span>
                <input
                  v-model.number="form.duration"
                  type="number"
                  min="15"
                  step="15"
                  :aria-invalid="Boolean(errors.duration)"
                />
                <small v-if="errors.duration" class="field-error">{{ errors.duration }}</small>
              </label>
            </div>

            <fieldset>
              <legend>Priorytet</legend>
              <div class="segmented-control">
                <label v-for="priority in ['Niski', 'Średni', 'Wysoki']" :key="priority">
                  <input v-model="form.priority" type="radio" :value="priority" />
                  <span>{{ priority }}</span>
                </label>
              </div>
            </fieldset>

            <div class="form-actions">
              <button type="submit" class="primary-button">Zapisz plan</button>
              <button type="button" class="secondary-button" @click="cancelForm">
                Anuluj
              </button>
            </div>
          </form>
        </div>

        <aside class="content-panel help-panel">
          <h2>Podpowiedzi</h2>
          <ul>
            <li>Używaj nazw przedmiotów znanych z planu zajęć.</li>
            <li>Wybierz realny termin, aby uniknąć planów z przeszłości.</li>
            <li>Podziel długie tematy na krótsze bloki po 45-60 minut.</li>
          </ul>
        </aside>
      </section>

      <section v-if="activeView === 'summary'" class="content-panel">
        <div class="section-title">
          <div>
            <p class="eyebrow">Podsumowanie</p>
            <h2>Zapisane plany</h2>
          </div>
          <button
            v-if="plans.length"
            type="button"
            class="danger-button"
            @click="clearAllPlans"
          >
            Wyczyść wszystko
          </button>
        </div>

        <div v-if="plans.length" class="plans-list">
          <article v-for="plan in sortedPlans" :key="plan.id" class="plan-card">
            <div>
              <div class="plan-card-header">
                <strong>{{ plan.subject }}</strong>
                <span class="badge">{{ plan.priority }}</span>
              </div>
              <p>{{ plan.topic }}</p>
              <small>{{ formatDate(plan.date) }} · {{ plan.duration }} min</small>
            </div>
            <button type="button" class="secondary-button" @click="removePlan(plan.id)">
              Usuń
            </button>
          </article>
        </div>

        <p v-else class="empty-state">
          Brak zapisanych planów. Przejdź do formularza i dodaj pierwszy wpis.
        </p>
      </section>
    </main>
  </div>
</template>
