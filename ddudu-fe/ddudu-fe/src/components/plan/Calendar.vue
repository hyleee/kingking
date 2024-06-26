<template>
  <div class="wrap col-flex jcc aic">
    <div class="flex aic" style="gap: 40px; margin-bottom: 20px;"> <!-- margin-bottom 추가 -->
      <button @click="addMonth(-1)">◀</button>
      <h1 class="cur-date">{{ year }}. {{ month }}.</h1>
      <button @click="addMonth(1)">▶</button>
    </div>
    <div class="calendar" ref="calendarRef">
      <div class="calendar-top">SUN</div>
      <div class="calendar-top">MON</div>
      <div class="calendar-top">TUE</div>
      <div class="calendar-top">WED</div>
      <div class="calendar-top">THU</div>
      <div class="calendar-top">FRI</div>
      <div class="calendar-top">SAT</div>
    </div>
    
    <ScheduleModal
      v-if="showModal"
      :year="year"
      :month="month"
      :day="day"
      @close="showModal = false"
    />
  </div>
  <Diary />
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';
import { useRouter } from 'vue-router';
import ScheduleModal from './ScheduleModal.vue';
import Diary from '@/components/plan/Diary.vue';
import { useScheduleManager } from '../composables/useScheduleManager';
import { useExerciseStore } from '@/stores/exerciseStore';

const props = defineProps({
  userId: String,
  exerciseDate: String
});

const { schedules, loadSchedule, refreshScheduleList } = useScheduleManager();

const today = new Date();
const year = ref(today.getFullYear());
const month = ref(today.getMonth() + 1);
const day = ref(null);
const showModal = ref(false);
const calendarRef = ref(null);

const router = useRouter();

const store = useExerciseStore();

const getFirstDay = (y, m) => {
  const date = new Date(`${y}-${m}-01`);
  return date.getDay();
};

const getLastDate = (y, m) => {
  const date = new Date(y, m, 0);
  return date.getDate();
};

const addMonth = (m) => {
  const date = new Date(year.value, month.value - 1 + m, 1);
  year.value = date.getFullYear();
  month.value = date.getMonth() + 1;
  showDates(year.value, month.value);
};

const showDates = (y, m) => {
  const before = calendarRef.value.querySelectorAll(".date");
  before.forEach(v => v.remove());

  const today = new Date();

  const todayDate =
    y == today.getFullYear() &&
    m == today.getMonth() + 1
      ? today.getDate()
      : null;

  for (let i = -getFirstDay(y, m) + 1; i <= getLastDate(y, m); i++) {
    const hasSchedule = schedules.value.find(v => v.date == `${y}-${m}-${i}`);
    const dateElement = document.createElement('div');
    dateElement.className = `date ${i < 1 ? "hidden-date" : ""} ${todayDate == i ? "today" : ""} ${hasSchedule ? "has-schedule" : ""}`;
    dateElement.innerHTML = `<p>${i}</p>`;
    dateElement.onclick = () => {
      day.value = i;
      const formattedDay = String(i).padStart(2, '0');
      const formattedMonth = String(month.value).padStart(2, '0');
      const date = `${year.value}-${formattedMonth}-${formattedDay}`;
      router.push({ name: 'plan', params: { userId: props.userId, exerciseDate: date } });
      store.setSelectedDate(date);
    };
    calendarRef.value.appendChild(dateElement);
  }
};

onMounted(() => {
  loadSchedule();
  showDates(year.value, month.value);
});

watch(() => props.exerciseDate, (newDate) => {
  const [newYear, newMonth] = newDate.split('-').map(Number);
  year.value = newYear;
  month.value = newMonth;
  showDates(newYear, newMonth);
});
</script>

<style scoped>
@import '@/assets/style.css';
</style>
