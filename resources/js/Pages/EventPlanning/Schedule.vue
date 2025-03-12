<script setup>
import { ref, onMounted, onUnmounted, reactive, computed, watch, h, defineComponent } from 'vue';
import { Users, Share2, Star, Video, UserCircle, Search, Plus, Mail, Phone, Upload, FileText, X, ChevronDown, ChevronUp, Pencil, PlusIcon, Trash2Icon, CalendarIcon, ClockIcon, UsersIcon, MapPinIcon, AlertCircle, ChevronLeft, ChevronRight, CheckSquare } from 'lucide-vue-next';

const Head = defineComponent({
  name: 'Project name',
  props: ['title'],
  render() { return null; }
});

// Component data
const isLiraOpen = ref(false);
const page = ref({
  projectDetails: {
    name: 'Sample Project'
  }
});

const view = ref('calendar');
const tabs = [
  { label: 'Calendar', value: 'calendar' },
  { label: 'List', value: 'list' },
  { label: 'Tasks', value: 'tasks' },
  { label: 'Timeline', value: 'timeline' }
];

const events = ref([
  {
    id: "1",
    title: "Venue Tour",
    description: "Initial venue walkthrough with client",
    date: "2025-03-15",
    startTime: "10:00",
    endTime: "11:30",
    location: "Grand Hall",
    assignedTo: ["Sarah", "Michael"],
    category: "Planning",
    tasks: [
      {
        id: "t1",
        title: "Prepare venue checklist",
        description: "Create a checklist of items to verify during the tour",
        dueDate: "2025-03-14",
        status: "Done",
        assignedTo: ["Sarah"]
      },
      {
        id: "t2",
        title: "Contact venue manager",
        description: "Confirm appointment with venue manager",
        dueDate: "2025-03-13",
        status: "Done",
        assignedTo: ["Michael"]
      }
    ]
  },
  {
    id: "2",
    title: "Vendor Meeting",
    description: "Meeting with catering team",
    date: "2025-03-18",
    startTime: "14:00",
    endTime: "15:00",
    location: "Office",
    assignedTo: ["Sarah"],
    category: "Vendor",
    tasks: [
      {
        id: "t3",
        title: "Prepare menu options",
        description: "Compile list of menu options to discuss",
        dueDate: "2025-03-17",
        status: "In Progress",
        assignedTo: ["Sarah"]
      }
    ]
  },
  {
    id: "3",
    title: "Client Review",
    description: "Review event progress with client",
    date: "2025-03-20",
    startTime: "11:00",
    endTime: "12:00",
    location: "Virtual",
    assignedTo: ["Michael", "Jessica"],
    category: "Client",
    tasks: [
      {
        id: "t4",
        title: "Prepare presentation",
        description: "Create slides for client review meeting",
        dueDate: "2025-03-19",
        status: "To Do",
        assignedTo: ["Jessica"]
      },
      {
        id: "t5",
        title: "Update budget spreadsheet",
        description: "Ensure budget is up to date before meeting",
        dueDate: "2025-03-19",
        status: "To Do",
        assignedTo: ["Michael"]
      }
    ]
  },
]);

const isAddEventOpen = ref(false);
const selectedDate = ref(null);
const selectedEvent = ref(null);
const showEventDetails = ref(false);
const collapsedEvents = ref({});
const taskFormRef = ref(null);
const showAddTaskForm = ref(false);
const editingTask = ref(null);

const errors = ref({
  title: '',
  date: '',
  startTime: '',
  endTime: '',
  location: '',
  category: ''
});

const newEvent = reactive({
  title: "",
  description: "",
  date: new Date().toISOString().split('T')[0],
  startTime: new Date().toLocaleTimeString('en-US', { hour12: false, hour: '2-digit', minute: '2-digit' }),
  endTime: new Date(Date.now() + 3600000).toLocaleTimeString('en-US', { hour12: false, hour: '2-digit', minute: '2-digit' }),
  location: "",
  assignedTo: [],
  category: "Planning",
  status: "Pending",
  tasks: []
});

const newTask = reactive({
  title: "",
  description: "",
  dueDate: "",
  assignedTo: [],
  status: "To Do"
});

const taskStatuses = ["To Do", "In Progress", "Done"];
const eventStatuses = ["Pending", "Confirmed", "Cancelled"];

const teamMembers = [
  {
    id: 1,
    name: "Sarah",
    avatar: "/placeholder.svg?height=40&width=40"
  },
  {
    id: 2,
    name: "Michael",
    avatar: "/placeholder.svg?height=40&width=40"
  },
  {
    id: 3,
    name: "Jessica",
    avatar: "/placeholder.svg?height=40&width=40"
  },
  {
    id: 4,
    name: "David",
    avatar: "/placeholder.svg?height=40&width=40"
  },
  {
    id: 5,
    name: "Emma",
    avatar: "/placeholder.svg?height=40&width=40"
  }
];

const categories = ["Planning", "Vendor", "Client", "Setup", "Event Day", "Post-Event"];
const weekdays = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];

const showModal = ref(false);
const showDropdown = ref(null);
const selectedFile = ref(null);
const showEditModal = ref(false);
const editingVendor = ref(null);

const newVendor = ref({
  name: '',
  company: '',
  email: '',
  phone: '',
  amount: '',
  status: 'Pending',
  documents: []
});

// Computed properties
const sortedEvents = computed(() => {
  return [...events.value].sort((a, b) => new Date(a.date).getTime() - new Date(b.date).getTime());
});

const timelineEvents = computed(() => {
  return [...events.value].sort((a, b) => {
    const dateA = new Date(a.date).getTime();
    const dateB = new Date(b.date).getTime();
    return dateA - dateB || a.startTime.localeCompare(b.startTime);
  });
});

const eventsByDate = computed(() => {
  return events.value.reduce((acc, event) => {
    if (!acc[event.date]) {
      acc[event.date] = [];
    }
    acc[event.date].push(event);
    return acc;
  }, {});
});

const eventsByCategory = computed(() => {
  return events.value.reduce((acc, event) => {
    if (!acc[event.category]) {
      acc[event.category] = [];
    }
    acc[event.category].push(event);
    return acc;
  }, {});
});

const currentDate = ref(new Date());

// Month/Year selector state
const showMonthSelector = ref(false);
const showYearSelector = ref(false);

// Months array
const months = [
  'January', 'February', 'March', 'April', 'May', 'June', 
  'July', 'August', 'September', 'October', 'November', 'December'
];

// Generate years array (20 years back, 20 years forward)
const generateYears = () => {
  const currentYear = currentDate.value.getFullYear();
  const years = [];
  for (let i = currentYear - 20; i <= currentYear + 20; i++) {
    years.push(i);
  }
  return years;
};

const years = computed(() => generateYears());

// Computed properties for current month and year
const currentMonth = computed(() => {
  return months[currentDate.value.getMonth()];
});

const currentYear = computed(() => {
  return currentDate.value.getFullYear();
});

// Toggle month selector
const toggleMonthSelector = () => {
  showMonthSelector.value = !showMonthSelector.value;
  if (showMonthSelector.value) {
    showYearSelector.value = false;
  }
};

// Toggle year selector
const toggleYearSelector = () => {
  showYearSelector.value = !showYearSelector.value;
  if (showYearSelector.value) {
    showMonthSelector.value = false;
  }
};

// Select month
const selectMonth = (monthIndex) => {
  currentDate.value = new Date(currentDate.value.getFullYear(), monthIndex, 1);
  showMonthSelector.value = false;
};

// Select year
const selectYear = (year) => {
  currentDate.value = new Date(year, currentDate.value.getMonth(), 1);
  showYearSelector.value = false;
};

// Close month/year selectors when clicking outside
const closeMonthYearSelectors = (e) => {
  if (showMonthSelector.value && !e.target.closest('.month-selector') && !e.target.closest('.month-display')) {
    showMonthSelector.value = false;
  }
  if (showYearSelector.value && !e.target.closest('.year-selector') && !e.target.closest('.year-display')) {
    showYearSelector.value = false;
  }
};

const currentMonthYear = computed(() => {
  const options = { month: 'long', year: 'numeric' };
  return currentDate.value.toLocaleDateString('en-US', options);
});

const goToPreviousMonth = () => {
  currentDate.value = new Date(currentDate.value.getFullYear(), currentDate.value.getMonth() - 1, 1);
};

const goToNextMonth = () => {
  currentDate.value = new Date(currentDate.value.getFullYear(), currentDate.value.getMonth() + 1, 1);
};

const calendarDays = computed(() => {
  const year = currentDate.value.getFullYear();
  const month = currentDate.value.getMonth();
  
  const daysInMonth = new Date(year, month + 1, 0).getDate();
  const firstDayOfMonth = new Date(year, month, 1).getDay();
  
  const days = [];
  
  // Add empty cells for days before the first day of the month
  for (let i = 0; i < firstDayOfMonth; i++) {
    days.push(null);
  }
  
  // Add days of the month
  for (let i = 1; i <= daysInMonth; i++) {
    const date = `${year}-${String(month + 1).padStart(2, '0')}-${String(i).padStart(2, '0')}`;
    days.push({
      day: i,
      date,
      events: eventsByDate.value[date] || [],
      isToday: isToday(year, month, i)
    });
  }
  
  return days;
});

const isToday = (year, month, day) => {
  const today = new Date();
  return today.getDate() === day && 
         today.getMonth() === month && 
         today.getFullYear() === year;
};

// Methods
function validateEventForm() {
  let isValid = true;
  const now = new Date();
  const selectedDateTime = new Date(`${newEvent.date}T${newEvent.startTime}`);

  // Clear errors for valid fields
  if (newEvent.title.trim()) {
    errors.value.title = '';
  } else {
    errors.value.title = 'Title is required';
    isValid = false;
  }

  if (newEvent.date) {
    if (selectedDateTime < now) {
      errors.value.date = 'Cannot select a past date';
      isValid = false;
    } else {
      errors.value.date = '';
    }
  } else {
    errors.value.date = 'Date is required';
    isValid = false;
  }

  if (newEvent.startTime) {
    if (selectedDateTime < now) {
      errors.value.startTime = 'Cannot select a past time';
      isValid = false;
    } else {
      errors.value.startTime = '';
    }
  } else {
    errors.value.startTime = 'Start time is required';
    isValid = false;
  }

  if (newEvent.endTime) {
    const endDateTime = new Date(`${newEvent.date}T${newEvent.endTime}`);
    if (endDateTime <= selectedDateTime) {
      errors.value.endTime = 'End time must be after start time';
      isValid = false;
    } else {
      errors.value.endTime = '';
    }
  } else {
    errors.value.endTime = 'End time is required';
    isValid = false;
  }

  if (newEvent.location.trim()) {
    errors.value.location = '';
  } else {
    errors.value.location = 'Location is required';
    isValid = false;
  }

  return isValid;
}

// Add watchers for real-time validation
watch(() => newEvent.title, (newValue) => {
  if (newValue.trim()) {
    errors.value.title = '';
  }
});

watch(() => newEvent.date, (newValue) => {
  const selectedDateTime = new Date(`${newValue}T${newEvent.startTime}`);
  const now = new Date();
  if (selectedDateTime < now) {
    errors.value.date = 'Cannot select a past date';
  } else {
    errors.value.date = '';
  }
});

watch(() => newEvent.startTime, (newValue) => {
  if (newValue) {
    const selectedDateTime = new Date(`${newEvent.date}T${newValue}`);
    const now = new Date();
    if (selectedDateTime < now) {
      errors.value.startTime = 'Cannot select a past time';
    } else {
      errors.value.startTime = '';
    }
  }
});

watch(() => newEvent.endTime, (newValue) => {
  if (newValue && newEvent.startTime) {
    const startDateTime = new Date(`${newEvent.date}T${newEvent.startTime}`);
    const endDateTime = new Date(`${newEvent.date}T${newValue}`);
    if (endDateTime <= startDateTime) {
      errors.value.endTime = 'End time must be after start time';
    } else {
      errors.value.endTime = '';
    }
  }
});

watch(() => newEvent.location, (newValue) => {
  if (newValue.trim()) {
    errors.value.location = '';
  }
});

function handleAddEvent() {
  if (!validateEventForm()) {
    return;
  }

  const event = {
    id: Date.now().toString(),
    ...newEvent,
    tasks: []
  };
  events.value.push(event);
  isAddEventOpen.value = false;
  
  // Reset form
  Object.assign(newEvent, {
    title: "",
    description: "",
    date: new Date().toISOString().split('T')[0],
    startTime: new Date().toLocaleTimeString('en-US', { hour12: false, hour: '2-digit', minute: '2-digit' }),
    endTime: new Date(Date.now() + 3600000).toLocaleTimeString('en-US', { hour12: false, hour: '2-digit', minute: '2-digit' }),
    location: "",
    assignedTo: [],
    category: "Planning",
    status: "Pending"
  });
}

function handleEditEvent() {
  if (!validateEventForm()) {
    return;
  }

  const eventIndex = events.value.findIndex(e => e.id === newEvent.id);
  if (eventIndex !== -1) {
    events.value[eventIndex] = { ...newEvent };
  }
  
  isAddEventOpen.value = false;
  
  // Reset form after editing
  Object.assign(newEvent, {
    title: "",
    description: "",
    date: new Date().toISOString().split('T')[0],
    startTime: new Date().toLocaleTimeString('en-US', { hour12: false, hour: '2-digit', minute: '2-digit' }),
    endTime: new Date(Date.now() + 3600000).toLocaleTimeString('en-US', { hour12: false, hour: '2-digit', minute: '2-digit' }),
    location: "",
    assignedTo: [],
    category: "Planning",
    status: "Pending"
  });
}

function handleAddTask() {
  if (!newTask.title.trim() || !newTask.dueDate) {
    return;
  }

  const now = new Date();
  const dueDate = new Date(newTask.dueDate);
  if (dueDate < now) {
    return; // Don't allow past due dates
  }

  // Find the event we're adding a task to
  const eventToUpdate = selectedEvent.value || events.value.find(e => e.date === selectedDate.value);
  
  if (eventToUpdate) {
    if (!eventToUpdate.tasks) {
      eventToUpdate.tasks = [];
    }
    
    const task = {
      id: `t${Date.now()}`,
      title: newTask.title,
      description: newTask.description,
      dueDate: newTask.dueDate,
      assignedTo: [...newTask.assignedTo],
      status: newTask.status
    };
    
    eventToUpdate.tasks.push(task);
    
    // Reset form
    newTask.title = "";
    newTask.description = "";
    newTask.dueDate = "";
    newTask.assignedTo = [];
    newTask.status = "To Do";
    
    // Hide the form after adding
    showAddTaskForm.value = false;
  }
}

function handleUpdateTask() {
  if (!editingTask.value || !editingTask.value.title.trim() || !editingTask.value.dueDate) {
    return;
  }

  const event = events.value.find(e => e.id === editingTask.value.eventId);
  if (event && event.tasks) {
    const taskIndex = event.tasks.findIndex(t => t.id === editingTask.value.id);
    if (taskIndex !== -1) {
      // Update the task
      event.tasks[taskIndex] = {
        id: editingTask.value.id,
        title: editingTask.value.title,
        description: editingTask.value.description,
        dueDate: editingTask.value.dueDate,
        assignedTo: [...editingTask.value.assignedTo],
        status: editingTask.value.status
      };
    }
  }
  
  // Clear editing state
  editingTask.value = null;
}

function handleDeleteEvent(id) {
  events.value = events.value.filter(event => event.id !== id);
  if (selectedEvent.value && selectedEvent.value.id === id) {
    selectedEvent.value = null;
    showEventDetails.value = false;
  }
}

function handleDeleteTask(eventId, taskId) {
  const event = events.value.find(e => e.id === eventId);
  if (event && event.tasks) {
    event.tasks = event.tasks.filter(task => task.id !== taskId);
  }
  
  // If we're editing this task, clear the editing state
  if (editingTask.value && editingTask.value.id === taskId) {
    editingTask.value = null;
  }
}

function handleAssigneeChange(member) {
  if (newEvent.assignedTo.includes(member)) {
    newEvent.assignedTo = newEvent.assignedTo.filter(name => name !== member);
  } else {
    newEvent.assignedTo.push(member);
  }
}

function handleTaskAssigneeChange(member) {
  if (newTask.assignedTo.includes(member)) {
    newTask.assignedTo = newTask.assignedTo.filter(name => name !== member);
  } else {
    newTask.assignedTo.push(member);
  }
}

function handleEditTaskAssigneeChange(member) {
  if (!editingTask.value) return;
  
  if (editingTask.value.assignedTo.includes(member)) {
    editingTask.value.assignedTo = editingTask.value.assignedTo.filter(name => name !== member);
  } else {
    editingTask.value.assignedTo.push(member);
  }
}

function getCategoryClass(category) {
  switch (category) {
    case "Planning":
      return "bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-300";
    case "Vendor":
      return "bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-300";
    case "Client":
      return "bg-purple-100 text-purple-800 dark:bg-purple-900 dark:text-purple-300";
    case "Setup":
      return "bg-orange-100 text-orange-800 dark:bg-orange-900 dark:text-orange-300";
    case "Event Day":
      return "bg-red-100 text-red-800 dark:bg-red-900 dark:text-red-300";
    default:
      return "bg-gray-100 text-gray-800 dark:bg-gray-800 dark:text-gray-300";
  }
}

function getStatusClass(status) {
  switch (status) {
    case "Done":
      return "bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-300";
    case "In Progress":
      return "bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-300";
    case "To Do":
      return "bg-gray-100 text-gray-800 dark:bg-gray-800 dark:text-gray-300";
    default:
      return "bg-gray-100 text-gray-800 dark:bg-gray-800 dark:text-gray-300";
  }
}

function formatDate(dateString) {
  return new Date(dateString).toLocaleDateString('en-US', {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  });
}

function showDayDetails(date) {
  selectedDate.value = date;
  selectedEvent.value = null;
  showEventDetails.value = true;
  showAddTaskForm.value = false;
  editingTask.value = null;
}

function showEventDetailsModal(event) {
  selectedEvent.value = event;
  selectedDate.value = event.date;
  showEventDetails.value = true;
  showAddTaskForm.value = false;
  editingTask.value = null;
}

function toggleEventCollapse(eventId) {
  collapsedEvents.value[eventId] = !collapsedEvents.value[eventId];
}

function scrollToTaskForm() {
  // Set the due date to the event date
  newTask.dueDate = selectedEvent.value.date;
  
  // Show the form
  showAddTaskForm.value = true;
  editingTask.value = null;
  
  // Wait for the next DOM update cycle
  setTimeout(() => {
    if (taskFormRef.value) {
      taskFormRef.value.scrollIntoView({ behavior: 'smooth', block: 'start' });
      // Focus on the first input field
      const firstInput = taskFormRef.value.querySelector('input');
      if (firstInput) {
        firstInput.focus();
      }
    }
  }, 100);
}

function editTask(eventId, task) {
  // Cancel any current editing
  if (editingTask.value) {
    editingTask.value = null;
  }
  
  // Set up editing for this task
  editingTask.value = {
    eventId,
    id: task.id,
    title: task.title,
    description: task.description,
    dueDate: task.dueDate,
    assignedTo: [...task.assignedTo],
    status: task.status
  };
  
  // Hide the add task form
  showAddTaskForm.value = false;
  
  // Scroll to the edit form
  setTimeout(() => {
    const editForm = document.getElementById(`edit-task-${task.id}`);
    if (editForm) {
      editForm.scrollIntoView({ behavior: 'smooth', block: 'start' });
    }
  }, 100);
}

function cancelEditTask() {
  editingTask.value = null;
}

function updateTaskStatus(eventId, taskId, newStatus) {
  const event = events.value.find(e => e.id === eventId);
  if (event && event.tasks) {
    const task = event.tasks.find(t => t.id === taskId);
    if (task) {
      task.status = newStatus;
    }
  }
}

// Close dropdowns handler
const closeDropdowns = (e) => {
  if (!e.target.closest('.status-dropdown')) {
    showDropdown.value = null;
  }
  
  // Also handle month/year selectors
  closeMonthYearSelectors(e);
};

onMounted(() => {
  document.addEventListener('click', closeDropdowns);
});

onUnmounted(() => {
  document.removeEventListener('click', closeDropdowns);
});
</script>

<template>
        <EventSidebar />
        <Header />
  <div class="min-h-screen overflow-y-auto">
    <div class="ml-64 pt-16">
      <div class="p-6">
        <div class="flex items-center justify-between mb-6">
          <div class="flex items-center gap-4">
            <h1 class="text-2xl font-bold">{{ page.projectDetails.name }}<span class="text-xl font-normal"> > Schedule</span></h1>
            <div class="flex items-center -space-x-2">
              <button class="p-2 text-gray-600 hover:text-gray-800">
                <Users class="w-5 h-5" />
              </button>
            </div>
          </div>
          <div class="flex items-center gap-4">
            <button><Share2 /></button>
            <button><Star /></button>
            <button>
              <Video />
            </button>
          </div>
        </div>

        <!-- Scheduler Component -->
        <div class="space-y-4">
          <div class="flex items-center justify-between"> 
            <div class="flex items-center gap-2">
              <!-- Tabs -->
              <div class="border rounded-md overflow-hidden">
                <div class="flex">
                  <button 
                    v-for="tab in tabs" 
                    :key="tab.value" 
                    @click="view = tab.value" 
                    class="px-3 py-2 text-sm font-medium"
                    :class="view === tab.value ? 'bg-blue-600 text-white' : 'hover:bg-muted'"
                  >
                    {{ tab.label }}
                  </button>
                </div>
              </div>
              
              <!-- Add Event Button -->
              <button 
                @click="isAddEventOpen = true" 
                class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 bg-blue-600 text-white hover:bg-blue-700 h-10 px-4 py-2"
              >
                <PlusIcon class="mr-2 h-4 w-4" />
                Add Event
              </button>
            </div>
          </div>

          <!-- Calendar View -->
          <div v-if="view === 'calendar'" class="border rounded-lg shadow-sm bg-card text-card-foreground">
            <div class="p-6">
              <!-- Month Navigation with Enhanced Month/Year Selector -->
              <div class="flex items-center justify-between mb-6">
                <button 
                  @click="goToPreviousMonth" 
                  class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 w-10 p-2"
                  title="Previous month"
                >
                  <ChevronLeft class="w-5 h-5" />
                </button>
                
                <!-- Centered Month/Year Display -->
                <div class="flex-1 flex justify-center items-center relative">
                  <div class="flex items-center gap-2">
                    <!-- Month Display (Clickable) -->
                    <span 
                      @click="toggleMonthSelector"
                      class="text-xl font-semibold cursor-pointer hover:text-blue-600 transition-colors month-display"
                    >
                      {{ currentMonth }}
                    </span>
                    
                    <!-- Year Display (Clickable) -->
                    <span 
                      @click="toggleYearSelector"
                      class="text-xl font-semibold cursor-pointer hover:text-blue-600 transition-colors year-display"
                    >
                      {{ currentYear }}
                    </span>
                  </div>
                  
                  <!-- Month Selector Dropdown (Vertical) -->
                  <div 
                    v-if="showMonthSelector" 
                    class="absolute top-full left-1/2 transform -translate-x-1/2 mt-1 bg-white border rounded-lg shadow-lg z-10 p-2 month-selector"
                    style="max-height: 300px; overflow-y: auto; width: 150px;"
                  >
                    <div class="flex flex-col">
                      <button
                        v-for="(month, index) in months"
                        :key="month"
                        @click="selectMonth(index)"
                        class="px-3 py-2 text-sm rounded-md transition-colors text-center"
                        :class="currentDate.getMonth() === index 
                          ? 'bg-blue-600 text-white' 
                          : 'hover:bg-gray-100'"
                      >
                        {{ month }}
                      </button>
                    </div>
                  </div>
                  
                  <!-- Year Selector Dropdown (Scrollable) -->
                  <div 
                    v-if="showYearSelector" 
                    class="absolute top-full left-1/2 transform -translate-x-1/2 mt-1 bg-white border rounded-lg shadow-lg z-10 p-2 year-selector"
                    style="max-height: 300px; overflow-y: auto; width: 120px;"
                  >
                    <div class="flex flex-col">
                      <button
                        v-for="year in years"
                        :key="year"
                        @click="selectYear(year)"
                        class="px-3 py-2 text-sm rounded-md transition-colors text-center"
                        :class="currentYear === year 
                          ? 'bg-blue-600 text-white' 
                          : 'hover:bg-gray-100'"
                      >
                        {{ year }}
                      </button>
                    </div>
                  </div>
                </div>
                
                <button 
                  @click="goToNextMonth" 
                  class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 w-10 p-2"
                  title="Next month"
                >
                  <ChevronRight class="w-5 h-5" />
                </button>
              </div>

              <!-- Calendar Grid -->
              <div class="grid grid-cols-7 gap-1">
                <div v-for="day in weekdays" :key="day" class="text-center font-medium py-2">
                  {{ day }}
                </div>
                <div
                  v-for="(day, index) in calendarDays"
                  :key="index"
                  :class="[
                    'min-h-[120px] border rounded-md p-1',
                    day ? 'bg-background cursor-pointer' : 'bg-muted/30',
                    day?.isToday ? 'border-blue-600 border-2' : '',
                    day && day.events.length > 0 ? 'hover:bg-gray-50' : ''
                  ]"
                  @click="day && day.events.length > 0 ? showDayDetails(day.date) : null"
                >
                  <template v-if="day">
                    <div class="text-sm font-medium p-1" :class="{ 'text-blue-600': day.isToday }">
                      {{ day.day }}
                    </div>
                    <div class="h-[90px] overflow-auto">
                      <div
                        v-for="event in day.events"
                        :key="event.id"
                        :class="`mb-1 p-1 text-xs rounded truncate ${getCategoryClass(event.category)}`"
                      >
                        {{ event.startTime }} - {{ event.title }}
                      </div>
                    </div>
                  </template>
                </div>
              </div>
            </div>
          </div>

          <!-- List View -->
          <div v-if="view === 'list'" class="border rounded-lg shadow-sm bg-card text-card-foreground">
            <div class="p-6">
              <div class="space-y-4">
                <div v-if="sortedEvents.length === 0" class="text-center py-8 text-muted-foreground">
                  No events scheduled. Click "Add Event" to create one.
                </div>
                <div 
                  v-for="event in sortedEvents" 
                  :key="event.id" 
                  class="border rounded-lg shadow-sm bg-card text-card-foreground"
                >
                  <div class="p-6 pb-2">
                    <div class="flex justify-between items-start">
                      <div>
                        <h3 class="text-lg font-semibold">{{ event.title }}</h3>
                        <div>
                          <span :class="`inline-block px-2 py-1 text-xs rounded-full mr-2 ${getCategoryClass(event.category)}`">
                            {{ event.category }}
                          </span>
                        </div>
                      </div>
                      <div class="flex gap-2">
                        <button 
                          @click="showEventDetailsModal(event)"
                          class="inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 w-10"
                        >
                          <Search class="h-4 w-4" />
                        </button>
                        <button 
                          @click="handleDeleteEvent(event.id)" 
                          class="inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 w-10"
                        >
                          <Trash2Icon class="h-4 w-4" />
                        </button>
                      </div>
                    </div>
                  </div>
                  <div class="p-6 pt-0">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                      <div class="space-y-2">
                        <div class="flex items-center text-sm">
                          <CalendarIcon class="mr-2 h-4 w-4 text-muted-foreground" />
                          <span>
                            {{ formatDate(event.date) }}
                          </span>
                        </div>
                        <div class="flex items-center text-sm">
                          <ClockIcon class="mr-2 h-4 w-4 text-muted-foreground" />
                          <span>{{ event.startTime }} - {{ event.endTime }}</span>
                        </div>
                        <div class="flex items-center text-sm">
                          <UsersIcon class="mr-2 h-4 w-4 text-muted-foreground" />
                          <span>{{ event.assignedTo.join(", ") }}</span>
                        </div>
                      </div>
                      <div>
                        <p class="text-sm">{{ event.description }}</p>
                        <p v-if="event.location" class="text-sm mt-2">
                          <span class="font-medium">Location:</span> {{ event.location }}
                        </p>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Tasks View -->
          <div v-if="view === 'tasks'" class="border rounded-lg shadow-sm bg-card text-card-foreground">
            <div class="p-6">
              <div v-if="Object.keys(eventsByCategory).length === 0" class="text-center py-8 text-muted-foreground">
                No events scheduled. Click "Add Event" to create one.
              </div>
              <div v-else class="space-y-6">
                <div v-for="(events, category) in eventsByCategory" :key="category" class="border rounded-lg p-4">
                  <h3 class="text-lg font-semibold mb-4 px-2 py-1 inline-block rounded" :class="getCategoryClass(category)">
                    {{ category }}
                  </h3>
                  
                  <div class="space-y-4">
                    <div v-for="event in events" :key="event.id" class="border rounded-lg shadow-sm">
                      <div class="p-4 border-b flex justify-between items-center cursor-pointer" @click="toggleEventCollapse(event.id)">
                        <div class="flex items-center gap-2">
                          <h4 class="font-medium">{{ event.title }}</h4>
                          <span class="text-xs text-gray-500">{{ event.date }} • {{ event.startTime }}</span>
                        </div>
                        <button class="p-1 hover:bg-gray-100 rounded">
                          <ChevronUp v-if="collapsedEvents[event.id]" class="w-5 h-5" />
                          <ChevronDown v-else class="w-5 h-5" />
                        </button>
                      </div>
                      
                      <div v-if="!collapsedEvents[event.id]" class="p-4">
                        <div v-if="!event.tasks || event.tasks.length === 0" class="text-sm text-gray-500 italic">
                          No tasks for this event
                        </div>
                        <div v-else class="space-y-3">
                          <div 
                            v-for="task in event.tasks" 
                            :key="task.id" 
                            class="border rounded p-3"
                          >
                            <div v-if="editingTask && editingTask.id === task.id" :id="`edit-task-${task.id}`" class="space-y-3">
                              <div class="flex justify-between items-center">
                                <h5 class="font-medium">Edit Task</h5>
                                <button @click.stop="cancelEditTask" class="text-gray-400 hover:text-gray-600">
                                  <X class="h-4 w-4" />
                                </button>
                              </div>
                              
                              <div>
                                <label class="block text-sm font-medium text-gray-700 mb-1">Task Title</label>
                                <input
                                  type="text"
                                  v-model="editingTask.title"
                                  class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                  placeholder="Task title"
                                />
                              </div>
                              
                              <div class="grid grid-cols-2 gap-4">
                                <div>
                                  <label class="block text-sm font-medium text-gray-700 mb-1">Due Date</label>
                                  <input
                                    type="date"
                                    v-model="editingTask.dueDate"
                                    class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                  />
                                </div>
                                <div>
                                  <label class="block text-sm font-medium text-gray-700 mb-1">Status</label>
                                  <select
                                    v-model="editingTask.status"
                                    class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                  >
                                    <option v-for="status in taskStatuses" :key="status" :value="status">
                                      {{ status }}
                                    </option>
                                  </select>
                                </div>
                              </div>
                              
                              <div>
                                <label class="block text-sm font-medium text-gray-700 mb-1">Description</label>
                                <textarea
                                  v-model="editingTask.description"
                                  rows="2"
                                  class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                  placeholder="Task description"
                                ></textarea>
                              </div>
                              
                              <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Assign To</label>
                                <div class="flex flex-wrap gap-2">
                                  <button
                                    v-for="member in teamMembers"
                                    :key="member.id"
                                    @click.stop="handleEditTaskAssigneeChange(member.name)"
                                    class="flex items-center gap-2 px-2 py-1 rounded border"
                                    :class="[
                                      editingTask.assignedTo.includes(member.name)
                                        ? 'bg-blue-600 text-white border-transparent'
                                        : 'border-gray-200 hover:border-gray-300'
                                    ]"
                                  >
                                    <img 
                                      :src="member.avatar" 
                                      :alt="member.name"
                                      class="w-5 h-5 rounded-full object-cover"
                                    />
                                    <span class="text-sm">{{ member.name }}</span>
                                  </button>
                                </div>
                              </div>
                              
                              <div class="flex justify-end gap-2 mt-4">
                                <button 
                                  @click.stop="cancelEditTask"
                                  class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-9 px-4 py-2"
                                >
                                  Cancel
                                </button>
                                <button 
                                  @click.stop="handleUpdateTask"
                                  class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 bg-blue-600 text-white hover:bg-blue-700 h-9 px-4 py-2"
                                >
                                  Update Task
                                </button>
                              </div>
                            </div>
                            
                            <div v-else class="flex justify-between items-start">
                              <div class="flex items-start gap-2">
                                <div class="relative">
                                  <select 
                                    v-model="task.status" 
                                    class="absolute opacity-0 w-full h-full cursor-pointer"
                                    @change="updateTaskStatus(event.id, task.id, task.status)"
                                  >
                                    <option v-for="status in taskStatuses" :key="status" :value="status">{{ status }}</option>
                                  </select>
                                  <CheckSquare class="w-4 h-4 mt-1 flex-shrink-0 cursor-pointer" :class="task.status === 'Done' ? 'text-green-500' : 'text-gray-400'" />
                                </div>
                                <div>
                                  <div class="flex items-center gap-2">
                                    <h5 class="font-medium" :class="task.status === 'Done' ? 'line-through text-gray-500' : ''">{{ task.title }}</h5>
                                    <span 
                                      :class="`text-xs px-2 py-0.5 rounded-full cursor-pointer ${getStatusClass(task.status)}`"
                                      @click.stop
                                    >
                                      <select 
                                        v-model="task.status" 
                                        class="bg-transparent border-0 cursor-pointer text-xs focus:ring-0 focus:outline-none w-full"
                                        @change="updateTaskStatus(event.id, task.id, task.status)"
                                        style="padding: 0; margin: 0;"
                                      >
                                        <option v-for="status in taskStatuses" :key="status" :value="status">{{ status }}</option>
                                      </select>
                                    </span>
                                  </div>
                                  <p class="text-sm text-gray-600">{{ task.description }}</p>
                                  <div class="flex items-center gap-4 mt-2 text-xs text-gray-500">
                                    <span>Due: {{ new Date(task.dueDate).toLocaleDateString() }}</span>
                                    <span v-if="task.assignedTo && task.assignedTo.length">
                                      Assigned to: {{ task.assignedTo.join(', ') }}
                                    </span>
                                  </div>
                                </div>
                              </div>
                              <div class="flex gap-2">
                                <button 
                                  @click.stop="editTask(event.id, task)" 
                                  class="text-gray-400 hover:text-blue-500"
                                >
                                  <Pencil class="h-4 w-4" />
                                </button>
                                <button 
                                  @click.stop="handleDeleteTask(event.id, task.id)" 
                                  class="text-gray-400 hover:text-red-500"
                                >
                                  <Trash2Icon class="h-4 w-4" />
                                </button>
                              </div>
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Timeline View -->
          <div v-if="view === 'timeline'" class="border rounded-lg shadow-sm bg-card text-card-foreground">
            <div class="p-6">
              <div class="relative">
                <div v-if="events.length === 0" class="text-center py-8 text-muted-foreground">
                  No events scheduled. Click "Add Event" to create one.
                </div>
                <div v-else class="border-l-2 border-blue-600/20 ml-6 pl-6 space-y-8">
                  <div 
                    v-for="event in timelineEvents" 
                    :key="event.id" 
                    class="relative"
                  >
                    <div class="absolute -left-10 w-4 h-4 rounded-full bg-blue-600"></div>
                    <div class="mb-1 flex items-center justify-between">
                      <h3 class="text-lg font-semibold">{{ event.title }}</h3>
                      <div class="flex gap-2">
                        <button 
                          @click="toggleEventCollapse(event.id)"
                          class="inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 w-10"
                        >
                          <ChevronUp v-if="collapsedEvents[event.id]" class="h-4 w-4" />
                          <ChevronDown v-else class="h-4 w-4" />
                        </button>
                        <button 
                          @click="handleDeleteEvent(event.id)" 
                          class="inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 w-10"
                        >
                          <Trash2Icon class="h-4 w-4" />
                        </button>
                      </div>
                    </div>
                    <div class="flex items-center gap-2 mb-2">
                      <span :class="`inline-block px-2 py-1 text-xs rounded-full ${getCategoryClass(event.category)}`">
                        {{ event.category }}
                      </span>
                      <span class="text-sm text-muted-foreground">
                        {{ new Date(event.date).toLocaleDateString() }} • {{ event.startTime }} - {{ event.endTime }}
                      </span>
                    </div>
                    <p class="text-sm mb-2">{{ event.description }}</p>
                    <div class="flex flex-wrap gap-2 items-center mb-3">
                      <div class="flex items-center text-sm">
                        <span class="font-medium mr-1">Location:</span> {{ event.location }}
                      </div>
                      <div class="flex items-center text-sm">
                        <span class="font-medium mr-1">Team:</span> {{ event.assignedTo.join(", ") }}
                      </div>
                    </div>
                    
                    <!-- Tasks for this event -->
                    <div v-if="!collapsedEvents[event.id] && event.tasks && event.tasks.length > 0" class="mt-4 pl-4 border-l-2 border-gray-200">
                      <h4 class="text-sm font-medium mb-2 flex items-center">
                        <CheckSquare class="w-4 h-4 mr-2" />
                        Tasks
                      </h4>
                      <div class="space-y-2">
                        <div 
                          v-for="task in event.tasks" 
                          :key="task.id"
                          class="p-2 border rounded-md bg-gray-50"
                          @click.stop="editTask(event.id, task)"
                        >
                          <div class="flex justify-between items-start">
                            <div class="flex items-start gap-2">
                              <div class="relative">
                                <select 
                                  v-model="task.status" 
                                  class="absolute opacity-0 w-full h-full cursor-pointer"
                                  @change="updateTaskStatus(event.id, task.id, task.status)"
                                >
                                  <option v-for="status in taskStatuses" :key="status" :value="status">{{ status }}</option>
                                </select>
                                <CheckSquare class="w-4 h-4 mt-1 flex-shrink-0 cursor-pointer" :class="task.status === 'Done' ? 'text-green-500' : 'text-gray-400'" />
                              </div>
                              <div>
                                <div class="flex items-center gap-2">
                                  <h5 class="text-sm font-medium" :class="task.status === 'Done' ? 'line-through text-gray-500' : ''">{{ task.title }}</h5>
                                  <span 
                                    :class="`text-xs px-2 py-0.5 rounded-full cursor-pointer ${getStatusClass(task.status)}`"
                                    @click.stop
                                  >
                                    <select 
                                      v-model="task.status" 
                                      class="bg-transparent border-0 cursor-pointer text-xs focus:ring-0 focus:outline-none w-full"
                                      @change="updateTaskStatus(event.id, task.id, task.status)"
                                      style="padding: 0; margin: 0;"
                                    >
                                      <option v-for="status in taskStatuses" :key="status" :value="status">{{ status }}</option>
                                    </select>
                                  </span>
                                </div>
                                <p class="text-xs text-gray-600">{{ task.description }}</p>
                                <div class="flex items-center gap-4 mt-1 text-xs text-gray-500">
                                  <span>Due: {{ new Date(task.dueDate).toLocaleDateString() }}</span>
                                </div>
                              </div>
                            </div>
                            <button 
                              @click.stop="handleDeleteTask(event.id, task.id)" 
                              class="text-gray-400 hover:text-red-500"
                            >
                              <Trash2Icon class="h-3 w-3" />
                            </button>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Event Details Modal -->
          <Teleport to="body">
            <Transition name="modal">
              <div v-if="showEventDetails" class="fixed inset-0 z-50 overflow-hidden">
                <!-- Backdrop -->
                <div class="absolute inset-0 bg-black bg-opacity-50" @click="showEventDetails = false"></div>
                
                <!-- Modal Container -->
                <div class="absolute inset-0 flex items-center justify-center p-4">
                  <div class="bg-white rounded-lg shadow-xl w-[90vw] max-w-3xl max-h-[90vh] flex flex-col">
                    <!-- Header -->
                    <div class="flex items-center justify-between px-6 py-4 border-b">
                      <div class="flex-1">
                        <h2 class="text-xl font-semibold">
                          {{ selectedEvent ? selectedEvent.title : formatDate(selectedDate) }}
                        </h2>
                      </div>
                      <button @click="showEventDetails = false" class="p-1 hover:bg-gray-100 rounded">
                        <X class="w-5 h-5" />
                      </button>
                    </div>

                    <!-- Content -->
                    <div class="flex-1 p-6 overflow-y-auto">
                      <div v-if="selectedEvent" class="space-y-6">
                        <!-- Event Details -->
                        <div class="space-y-4">
                          <div class="flex justify-between items-start">
                            <div class="flex items-center gap-2">
                              <span :class="`inline-block px-2 py-1 text-xs rounded-full ${getCategoryClass(selectedEvent.category)}`">
                                {{ selectedEvent.category }}
                              </span>
                              <span class="text-sm text-muted-foreground">
                                {{ selectedEvent.startTime }} - {{ selectedEvent.endTime }}
                              </span>
                            </div>
                            <button 
                              @click="isAddEventOpen = true; Object.assign(newEvent, {...selectedEvent})"
                              class="inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-9 px-3 py-2"
                            >
                              <Pencil class="h-4 w-4" />
                            </button>
                          </div>
                          
                          <p class="text-sm">{{ selectedEvent.description }}</p>
                          
                          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <div class="space-y-2">
                              <div class="flex items-center text-sm">
                                <MapPinIcon class="mr-2 h-4 w-4 text-muted-foreground" />
                                <span>{{ selectedEvent.location }}</span>
                              </div>
                              <div class="flex items-center text-sm">
                                <UsersIcon class="mr-2 h-4 w-4 text-muted-foreground" />
                                <span>{{ selectedEvent.assignedTo.join(", ") }}</span>
                              </div>
                            </div>
                          </div>
                        </div>
                        
                        <!-- Tasks Section -->
                        <div class="space-y-4">
                          <div class="flex items-center justify-between">
                            <h3 class="text-lg font-medium">Tasks</h3>
                            <button 
                              @click="scrollToTaskForm"
                              class="text-sm text-blue-600 hover:text-blue-800"
                            >
                              + Add Task
                            </button>
                          </div>
                          
                          <div v-if="!selectedEvent.tasks || selectedEvent.tasks.length === 0" class="text-sm text-gray-500 italic">
                            No tasks for this event
                          </div>
                          
                          <div v-else class="space-y-3">
                            <div 
                              v-for="task in selectedEvent.tasks" 
                              :key="task.id" 
                              class="border rounded p-3"
                              :class="{ 'bg-blue-50': editingTask && editingTask.id === task.id }"
                              @click="editingTask && editingTask.id !== task.id ? editTask(selectedEvent.id, task) : null"
                            >
                              <!-- Task Edit Form -->
                              <div v-if="editingTask && editingTask.id === task.id" :id="`edit-task-${task.id}`" class="space-y-3">
                                <div class="flex justify-between items-center">
                                  <h5 class="font-medium">Edit Task</h5>
                                  <button @click.stop="cancelEditTask" class="text-gray-400 hover:text-gray-600">
                                    <X class="h-4 w-4" />
                                  </button>
                                </div>
                                
                                <div>
                                  <label class="block text-sm font-medium text-gray-700 mb-1">Task Title</label>
                                  <input
                                    type="text"
                                    v-model="editingTask.title"
                                    class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                    placeholder="Task title"
                                  />
                                </div>
                                
                                <div class="grid grid-cols-2 gap-4">
                                  <div>
                                    <label class="block text-sm font-medium text-gray-700 mb-1">Due Date</label>
                                    <input
                                      type="date"
                                      v-model="editingTask.dueDate"
                                      class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                    />
                                  </div>
                                  <div>
                                    <label class="block text-sm font-medium text-gray-700 mb-1">Status</label>
                                    <select
                                      v-model="editingTask.status"
                                      class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                    >
                                      <option v-for="status in taskStatuses" :key="status" :value="status">
                                        {{ status }}
                                      </option>
                                    </select>
                                  </div>
                                </div>
                                
                                <div>
                                  <label class="block text-sm font-medium text-gray-700 mb-1">Description</label>
                                  <textarea
                                    v-model="editingTask.description"
                                    rows="2"
                                    class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                    placeholder="Task description"
                                  ></textarea>
                                </div>
                                
                                <div>
                                  <label class="block text-sm font-medium text-gray-700 mb-2">Assign To</label>
                                  <div class="flex flex-wrap gap-2">
                                    <button
                                      v-for="member in teamMembers"
                                      :key="member.id"
                                      @click.stop="handleEditTaskAssigneeChange(member.name)"
                                      class="flex items-center gap-2 px-2 py-1 rounded border"
                                      :class="[
                                        editingTask.assignedTo.includes(member.name)
                                          ? 'bg-blue-600 text-white border-transparent'
                                          : 'border-gray-200 hover:border-gray-300'
                                      ]"
                                    >
                                      <img 
                                        :src="member.avatar" 
                                        :alt="member.name"
                                        class="w-5 h-5 rounded-full object-cover"
                                      />
                                      <span class="text-sm">{{ member.name }}</span>
                                    </button>
                                  </div>
                                </div>
                                
                                <div class="flex justify-end gap-2 mt-4">
                                  <button 
                                    @click.stop="cancelEditTask"
                                    class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-9 px-4 py-2"
                                  >
                                    Cancel
                                  </button>
                                  <button 
                                    @click.stop="handleUpdateTask"
                                    class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 bg-blue-600 text-white hover:bg-blue-700 h-9 px-4 py-2"
                                  >
                                    Update Task
                                  </button>
                                </div>
                              </div>
                              
                              <!-- Task View -->
                              <div v-else class="flex justify-between items-start">
                                <div class="flex items-start gap-2">
                                  <div class="relative">
                                    <select 
                                      v-model="task.status" 
                                      class="absolute opacity-0 w-full h-full cursor-pointer"
                                      @change.stop="updateTaskStatus(selectedEvent.id, task.id, task.status)"
                                    >
                                      <option v-for="status in taskStatuses" :key="status" :value="status">{{ status }}</option>
                                    </select>
                                    <CheckSquare class="w-4 h-4 mt-1 flex-shrink-0 cursor-pointer" :class="task.status === 'Done' ? 'text-green-500' : 'text-gray-400'" />
                                  </div>
                                  <div>
                                    <div class="flex items-center gap-2">
                                      <h5 class="font-medium" :class="task.status === 'Done' ? 'line-through text-gray-500' : ''">{{ task.title }}</h5>
                                      <span :class="`text-xs px-2 py-0.5 rounded-full ${getStatusClass(task.status)}`">
                                        <select 
                                          v-model="task.status" 
                                          class="bg-transparent border-0 cursor-pointer text-xs focus:ring-0 focus:outline-none w-full"
                                          @change="updateTaskStatus(selectedEvent.id, task.id, task.status)"
                                          style="padding: 0; margin: 0;"
                                        >
                                          <option v-for="status in taskStatuses" :key="status" :value="status">{{ status }}</option>
                                        </select>
                                      </span>
                                    </div>
                                    <p class="text-sm text-gray-600">{{ task.description }}</p>
                                    <div class="flex items-center gap-4 mt-2 text-xs text-gray-500">
                                      <span>Due: {{ new Date(task.dueDate).toLocaleDateString() }}</span>
                                      <span v-if="task.assignedTo && task.assignedTo.length">
                                        Assigned to: {{ task.assignedTo.join(', ') }}
                                      </span>
                                    </div>
                                  </div>
                                </div>
                                <div class="flex gap-2">
                                  <button 
                                    @click.stop="editTask(selectedEvent.id, task)" 
                                    class="text-gray-400 hover:text-blue-500"
                                  >
                                    <Pencil class="h-4 w-4" />
                                  </button>
                                  <button 
                                    @click.stop="handleDeleteTask(selectedEvent.id, task.id)" 
                                    class="text-gray-400 hover:text-red-500"
                                  >
                                    <Trash2Icon class="h-4 w-4" />
                                  </button>
                                </div>
                              </div>
                            </div>
                          </div>
                        </div>
                        
                        <!-- Add Task Form -->
                        <div v-if="showAddTaskForm" ref="taskFormRef" class="border rounded-lg p-4 bg-gray-50">
                          <div class="flex justify-between items-center mb-3">
                            <h4 class="text-sm font-medium">Add New Task</h4>
                            <button @click="showAddTaskForm = false" class="text-gray-400 hover:text-gray-600">
                              <X class="h-4 w-4" />
                            </button>
                          </div>
                          <div class="space-y-4">
                            <div>
                              <label class="block text-sm font-medium text-gray-700 mb-1">Task Title</label>
                              <input
                                type="text"
                                v-model="newTask.title"
                                class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                placeholder="Task title"
                              />
                            </div>
                            
                            <div class="grid grid-cols-2 gap-4">
                              <div>
                                <label class="block text-sm font-medium text-gray-700 mb-1">Due Date</label>
                                <input
                                  type="date"
                                  v-model="newTask.dueDate"
                                  class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                />
                              </div>
                              <div>
                                <label class="block text-sm font-medium text-gray-700 mb-1">Status</label>
                                <select
                                  v-model="newTask.status"
                                  class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                >
                                  <option v-for="status in taskStatuses" :key="status" :value="status">
                                    {{ status }}
                                  </option>
                                </select>
                              </div>
                            </div>
                            
                            <div>
                              <label class="block text-sm font-medium text-gray-700 mb-1">Description</label>
                              <textarea
                                v-model="newTask.description"
                                rows="2"
                                class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                placeholder="Task description"
                              ></textarea>
                            </div>
                            
                            <div>
                              <label class="block text-sm font-medium text-gray-700 mb-2">Assign To</label>
                              <div class="flex flex-wrap gap-2">
                                <button
                                  v-for="member in teamMembers"
                                  :key="member.id"
                                  @click="handleTaskAssigneeChange(member.name)"
                                  class="flex items-center gap-2 px-2 py-1 rounded border"
                                  :class="[
                                    newTask.assignedTo.includes(member.name)
                                      ? 'bg-blue-600 text-white border-transparent'
                                      : 'border-gray-200 hover:border-gray-300'
                                  ]"
                                >
                                  <img 
                                    :src="member.avatar" 
                                    :alt="member.name"
                                    class="w-5 h-5 rounded-full object-cover"
                                  />
                                  <span class="text-sm">{{ member.name }}</span>
                                </button>
                              </div>
                            </div>
                            
                            <button 
                              @click="handleAddTask"
                              class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 bg-blue-600 text-white hover:bg-blue-700 h-10 px-4 py-2 w-full"
                              :class="{ 'opacity-50 cursor-not-allowed': !newTask.title || !newTask.dueDate }"
                              :disabled="!newTask.title || !newTask.dueDate"
                            >
                              Add Task
                            </button>
                          </div>
                        </div>
                      </div>
                      
                      <!-- Day Events View -->
                      <div v-else-if="selectedDate" class="space-y-6">
                        <h3 class="text-lg font-medium">Events</h3>
                        
                        <div v-if="!eventsByDate[selectedDate] || eventsByDate[selectedDate].length === 0" class="text-sm text-gray-500 italic">
                          No events scheduled for this day
                        </div>
                        
                        <div v-else class="space-y-4">
                          <div 
                            v-for="event in eventsByDate[selectedDate]" 
                            :key="event.id" 
                            class="border rounded-lg p-4 hover:bg-gray-50 cursor-pointer"
                            @click="showEventDetailsModal(event)"
                          >
                            <div class="flex justify-between items-start">
                              <div>
                                <h4 class="font-medium">{{ event.title }}</h4>
                                <div class="flex items-center gap-2 mt-1">
                                  <span :class="`inline-block px-2 py-1 text-xs rounded-full ${getCategoryClass(event.category)}`">
                                    {{ event.category }}
                                  </span>
                                  <span class="text-sm text-gray-500">
                                    {{ event.startTime }} - {{ event.endTime }}
                                  </span>
                                </div>
                              </div>
                              <button 
                                @click.stop="handleDeleteEvent(event.id)" 
                                class="text-gray-400 hover:text-red-500"
                              >
                                <Trash2Icon class="h-4 w-4" />
                              </button>
                            </div>
                            <p class="text-sm text-gray-600 mt-2">{{ event.description }}</p>
                            <div class="mt-2 text-sm">
                              <span class="font-medium">Location:</span> {{ event.location }}
                            </div>
                            <div class="mt-1 text-sm">
                              <span class="font-medium">Team:</span> {{ event.assignedTo.join(", ") }}
                            </div>
                            
                            <!-- Tasks count -->
                            <div v-if="event.tasks && event.tasks.length > 0" class="mt-3 text-sm text-blue-600">
                              {{ event.tasks.length }} task{{ event.tasks.length > 1 ? 's' : '' }} associated
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>

                    <!-- Footer -->
                    <div class="px-6 py-4 border-t bg-gray-50 flex justify-between items-center">
                      <div class="text-sm text-gray-500">
                        Press Esc to close
                      </div>
                      <button 
                        @click="showEventDetails = false" 
                        class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 px-4 py-2"
                      >
                        Close
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </Transition>
          </Teleport>

          <!-- Add Event Dialog -->
          <Teleport to="body">
            <Transition name="modal">
              <div v-if="isAddEventOpen" class="fixed inset-0 z-50 overflow-hidden">
                <!-- Backdrop -->
                <div class="absolute inset-0 bg-black bg-opacity-50" @click="isAddEventOpen = false"></div>
                
                <!-- Modal Container -->
                <div class="absolute inset-0 flex items-center justify-center p-4">
                  <div class="bg-white rounded-lg shadow-xl w-[90vw] max-h-[90vh] flex flex-col">
                    <!-- Header -->
                    <div class="flex items-center justify-between px-6 py-4 border-b">
                      <div class="flex-1">
                        <input 
                          v-model="newEvent.title"
                          type="text"
                          class="text-xl font-semibold w-full bg-transparent border-0 focus:ring-0 focus:outline-none"
                          placeholder="Event title"
                          :class="{ 'border-red-500': errors.title }"
                        />
                        <div v-if="errors.title" class="text-sm text-red-500 flex items-center gap-1 mt-1">
                          <AlertCircle class="w-4 h-4" />
                          {{ errors.title }}
                        </div>
                      </div>
                      <button @click="isAddEventOpen = false" class="p-1 hover:bg-gray-100 rounded">
                        <X class="w-5 h-5" />
                      </button>
                    </div>

                    <!-- Content -->
                    <div class="flex flex-1 min-h-0">
                      <!-- Left Column - Event Details -->
                      <div class="w-2/3 border-r p-6 overflow-y-auto">
                        <div class="space-y-6">
                          <!-- Date, Time and Status Row -->
                          <div class="grid grid-cols-4 gap-4">
                            <div>
                              <label class="block text-sm font-medium text-gray-700 mb-1">Date</label>
                              <input
                                type="date"
                                v-model="newEvent.date"
                                class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                :class="{ 'border-red-500': errors.date }"
                              />
                              <div v-if="errors.date" class="text-sm text-red-500 mt-1">{{ errors.date }}</div>
                            </div>
                            <div>
                              <label class="block text-sm font-medium text-gray-700 mb-1">Start Time</label>
                              <input
                                type="time"
                                v-model="newEvent.startTime"
                                class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                :class="{ 'border-red-500': errors.startTime }"
                              />
                              <div v-if="errors.startTime" class="text-sm text-red-500 mt-1">{{ errors.startTime }}</div>
                            </div>
                            <div>
                              <label class="block text-sm font-medium text-gray-700 mb-1">End Time</label>
                              <input
                                type="time"
                                v-model="newEvent.endTime"
                                class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                :class="{ 'border-red-500': errors.endTime }"
                              />
                              <div v-if="errors.endTime" class="text-sm text-red-500 mt-1">{{ errors.endTime }}</div>
                            </div>
                            <div>
                              <label class="block text-sm font-medium text-gray-700 mb-1">Status</label>
                              <select
                                v-model="newEvent.status"
                                class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                              >
                                <option v-for="status in eventStatuses" :key="status" :value="status">
                                  {{ status }}
                                </option>
                              </select>
                            </div>
                          </div>

                          <!-- Category and Location Row -->
                          <div class="grid grid-cols-2 gap-4">
                            <div>
                              <label class="block text-sm font-medium text-gray-700 mb-1">Category</label>
                              <select
                                v-model="newEvent.category"
                                class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                :class="{ 'border-red-500': errors.category }"
                              >
                                <option v-for="category in categories" :key="category" :value="category">
                                  {{ category }}
                                </option>
                              </select>
                              <div v-if="errors.category" class="text-sm text-red-500 mt-1">{{ errors.category }}</div>
                            </div>
                            <div>
                              <label class="block text-sm font-medium text-gray-700 mb-1">Location</label>
                              <input
                                type="text"
                                v-model="newEvent.location"
                                class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                                :class="{ 'border-red-500': errors.location }"
                                placeholder="Event location"
                              />
                              <div v-if="errors.location" class="text-sm text-red-500 mt-1">{{ errors.location }}</div>
                            </div>
                          </div>

                          <!-- Description -->
                          <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Description</label>
                            <textarea 
                              v-model="newEvent.description"
                              rows="4"
                              class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                              placeholder="Add event description..."
                            ></textarea>
                          </div>

                          <!-- Team Members Assignment -->
                          <div>
                            <label class="block text-sm font-medium text-gray-700 mb-3">Team Members</label>
                            <div class="flex flex-wrap gap-3">
                              <button
                                v-for="member in teamMembers"
                                :key="member.id"
                                @click="handleAssigneeChange(member.name)"
                                class="flex items-center gap-2 px-3 py-2 rounded-lg border transition-colors"
                                :class="[
                                  newEvent.assignedTo.includes(member.name)
                                    ? 'bg-blue-600 text-white border-transparent'
                                    : 'border-gray-200 hover:border-gray-300'
                                ]"
                              >
                                <img 
                                  :src="member.avatar" 
                                  :alt="member.name"
                                  class="w-6 h-6 rounded-full object-cover"
                                />
                                <span>{{ member.name }}</span>
                              </button>
                            </div>
                          </div>
                        </div>
                      </div>

                      <!-- Right Column - Task Creation -->
                      <div class="w-1/3 p-6 overflow-y-auto bg-gray-50">
                        <div class="space-y-6">
                          <h3 class="text-lg font-medium">Create Related Task</h3>
                          
                          <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Task Title</label>
                            <input
                              type="text"
                              v-model="newTask.title"
                              class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                              placeholder="Task title"
                            />
                          </div>

                          <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Due Date</label>
                            <input
                              type="date"
                              v-model="newTask.dueDate"
                              class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                            />
                          </div>

                          <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Description</label>
                            <textarea
                              v-model="newTask.description"
                              rows="3"
                              class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                              placeholder="Task description"
                            ></textarea>
                          </div>

                          <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Status</label>
                            <select
                              v-model="newTask.status"
                              class="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"
                            >
                              <option v-for="status in taskStatuses" :key="status" :value="status">
                                {{ status }}
                              </option>
                            </select>
                          </div>

                          <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Assign To</label>
                            <div class="flex flex-wrap gap-2">
                              <button
                                v-for="member in teamMembers"
                                :key="member.id"
                                @click="handleTaskAssigneeChange(member.name)"
                                class="flex items-center gap-2 px-2 py-1 rounded border"
                                :class="[
                                  newTask.assignedTo.includes(member.name)
                                    ? 'bg-blue-600 text-white border-transparent'
                                    : 'border-gray-200 hover:border-gray-300'
                                ]"
                              >
                                <img 
                                  :src="member.avatar" 
                                  :alt="member.name"
                                  class="w-5 h-5 rounded-full object-cover"
                                />
                                <span class="text-sm">{{ member.name }}</span>
                              </button>
                            </div>
                          </div>

                          <button 
                            @click="handleAddTask"
                            class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 bg-blue-600 text-white hover:bg-blue-700 h-10 px-4 py-2 w-full"
                            :class="{ 'opacity-50 cursor-not-allowed': !newTask.title || !newTask.dueDate }"
                            :disabled="!newTask.title || !newTask.dueDate"
                          >
                            Add Task
                          </button>
                        </div>
                      </div>
                    </div>

                    <!-- Footer -->
                    <div class="px-6 py-4 border-t bg-gray-50 flex justify-between items-center">
                      <div class="text-sm text-gray-500">
                        Press Esc to close
                      </div>
                      <div class="flex gap-3">
                        <button 
                          @click="isAddEventOpen = false" 
                          class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 px-4 py-2"
                        >
                          Cancel
                        </button>
                        <button 
                          @click="newEvent.id ? handleEditEvent() : handleAddEvent()"
                          class="inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 bg-blue-600 text-white hover:bg-blue-700 h-10 px-4 py-2"
                          :class="{ 'opacity-50 cursor-not-allowed': !newEvent.title || !newEvent.date }"
                          :disabled="!newEvent.title || !newEvent.date"
                        >
                          {{ newEvent.id ? 'Update Event' : 'Create Event' }}
                        </button>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </Transition>
          </Teleport>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.text-muted-foreground {
  color: hsl(215.4 16.3% 56.9%);
}

.bg-card {
  background-color: hsl(0 0% 100%);
}

.text-card-foreground {
  color: hsl(222.2 47.4% 11.2%);
}

.bg-primary {
  background-color: #2563eb; /* blue-600 */
}

.text-primary-foreground {
  color: hsl(210 40% 98%);
}

.bg-muted {
  background-color: hsl(210 40% 96.1%);
}

.bg-background {
  background-color: hsl(0 0% 100%);
}

.border-input {
  border-color: hsl(214.3 31.8% 91.4%);
}

.bg-accent {
  background-color: hsl(210 40% 96.1%);
}

.text-accent-foreground {
  color: hsl(222.2 47.4% 11.2%);
}

.border-primary {
  border-color: #2563eb; /* blue-600 */
}

/* Month/Year selector styles */
.month-selector, .year-selector {
  animation: fadeIn 0.2s ease-in-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-10px) translateX(-50%); }
  to { opacity: 1; transform: translateY(0) translateX(-50%); }
}

/* Scrollbar styling */
::-webkit-scrollbar {
  width: 6px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb:hover {
  background: #a1a1a1;
}

@media (prefers-color-scheme: dark) {
  .dark\:bg-blue-900 {
    background-color: rgba(30, 58, 138, 1);
  }
  .dark\:text-blue-300 {
    color: rgba(147, 197, 253, 1);
  }
  .dark\:bg-green-900 {
    background-color: rgba(20, 83, 45, 1);
  }
  .dark\:text-green-300 {
    color: rgba(134, 239, 172, 1);
  }
  .dark\:bg-purple-900 {
    background-color: rgba(88, 28, 135, 1);
  }
  .dark\:text-purple-300 {
    color: rgba(216, 180, 254, 1);
  }
  .dark\:bg-orange-900 {
    background-color: rgba(124, 45, 18, 1);
  }
  .dark\:text-orange-300 {
    color: rgba(253, 186, 116, 1);
  }
  .dark\:bg-red-900 {
    background-color: rgba(127, 29, 29, 1);
  }
  .dark\:text-red-300 {
    color: rgba(252, 165, 165, 1);
  }
  .dark\:bg-gray-800 {
    background-color: rgba(31, 41, 55, 1);
  }
  .dark\:text-gray-300 {
    color: rgba(209, 213, 219, 1);
  }
}

.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.2s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}
</style>