<template>
    <div class="inbox-container">
        <h2>Inbox (<span>{{ unreadCount }}</span> unread)</h2>
        <ul v-if="notifications.length">
            <li v-for="notif in notifications" :key="notif._id" :class="{ read: notif.read }">
                <strong>{{ notif.subject || 'Notification' }}</strong><br />
                <span v-html="notif.body" /><br />
                <small>{{ formatDate(notif.createdAt) }}</small>
                <div class="actions">
                    <button v-if="!notif.read" @click="markAsRead(notif)">Mark Read</button>
                    <button @click="archive(notif)">Archive</button>
                </div>
            </li>
        </ul>
        <p v-else>No notifications</p>

        <hr />

        <h3>Notification Preferences</h3>
        <div v-if="preferences.length">
            <div v-for="pref, index) in filteredPreferences" :key="index" class="pref-row">
                <!-- {{ pref }} -->
                <template v-if="pref.level === 'global'">
                    <strong>Global</strong><br />
                </template>
                <template v-else>
                    <strong>{{ pref.workflow.name }}</strong><br />
                </template>

                <label>
                    <input type="checkbox" v-model="pref.channels.in_app" />
                    In-App
                </label>
                <label>
                    <input type="checkbox" v-model="pref.channels.email" />
                    Email
                </label>
            </div>
        </div>
        <p v-else>Loading preferences...</p>
    </div>
</template>

<script setup>
import { onMounted, computed, ref } from 'vue'
import { Novu } from '@novu/js'

const novu = new Novu({
    apiUrl: import.meta.env.VITE_APP_API_URL,
    socketUrl: import.meta.env.VITE_APP_SOCKET_URL,
    applicationIdentifier: import.meta.env.VITE_APP_APP_ID,
    subscriberId: import.meta.env.VITE_APP_SUBSCRIBER_ID
})

// const subscriberId = import.meta.env.VITE_APP_SUBSCRIBER_ID // Replace with real user ID

const notifications = ref([])
const unreadCount = ref(0)
const preferences = ref([])

// Load notifications
const loadNotifications = async () => {
    const { data } = await novu.notifications.list({
        limit: 20,
        // tags: ['customer-visible'], // Filter by workflow tag
    })

    // console.log(data)
    notifications.value = data.notifications
    unreadCount.value = data.notifications.filter((n) => !n.isRead).length
}

// Load preferences
const loadPreferences = async () => {
    const { data } = await novu.preferences.list()
    let filteredData = data.filter(i => { if (i.workflow === undefined) return true; return i.workflow.tags.includes("Public"); })
    preferences.value = filteredData
}

// Mark as read
const markAsRead = async (notif) => {
    await notif.read()
    loadNotifications()
}

// Archive
const archive = async (notif) => {
    await notif.archive()
    loadNotifications()
}

// Toggle channel (IN_APP / EMAIL)
// const toggleChannel = async (templateId, channel, enabled) => {
//     await novu.subscriber.updatePreference(templateId, { channel, enabled })
// }

// Filter customer-visible workflows
const filteredPreferences = computed(() =>
    preferences.value
    //   preferences.value.filter((p) =>
    //     p.template.tags?.includes('customer-visible')
    //   )
)

// Real-time listeners
onMounted(async () => {
    //   await novu.initialize({ subscriberId })

    novu.on('notifications.notification_received', (data) => {
        loadNotifications()
        console.log('New notification:', data);
    })

    novu.on('notifications.unread_count_changed', ({ count }) => {
        unreadCount.value = count
    })

    await loadNotifications()
    await loadPreferences()
})

// Helpers
const formatDate = (dateStr) => new Date(dateStr).toLocaleString()

</script>


<style scoped>
.inbox-container {
    max-width: 600px;
    margin: auto;
    font-family: sans-serif;
    background: #fff;
    padding: 20px;
    border-radius: 6px;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

ul {
    list-style: none;
    padding: 0;
}

li {
    border-bottom: 1px solid #ddd;
    padding: 12px 0;
}

li.read {
    opacity: 0.6;
}

.actions {
    margin-top: 5px;
}

button {
    background: #007bff;
    color: white;
    border: none;
    padding: 6px 10px;
    margin-right: 5px;
    border-radius: 4px;
    cursor: pointer;
}

button:hover {
    background: #0056b3;
}

.pref-row {
    margin-bottom: 12px;
    padding-bottom: 8px;
    border-bottom: 1px solid #ddd;
}

.pref-row label {
    margin-right: 20px;
}
</style>