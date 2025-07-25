<template>
    <div>
        <div ref="novuInbox">MOUNT ME!</div>
    </div>
</template>

<script setup>
import { useTemplateRef, onMounted, onUnmounted, ref } from "vue";
import { NovuUI } from "@novu/js/ui"

const novuInbox = useTemplateRef('novuInbox');
let novuInstance = null;

const onNotificationClick = (notification) => console.log(notification)
const renderNotification = (el, notification) => `<div><p>RENDERERED ELSEWHERE</p><p>${ notification.subject }</p<p>${ notification.body }</p></div>`
const iconAlien = `<svg width="800px" height="800px" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
<path fill-rule="evenodd" clip-rule="evenodd" d="M8 16L3.54223 12.3383C1.93278 11.0162 1 9.04287 1 6.96005C1 3.11612 4.15607 0 8 0C11.8439 0 15 3.11612 15 6.96005C15 9.04287 14.0672 11.0162 12.4578 12.3383L8 16ZM3 6H5C6.10457 6 7 6.89543 7 8V9L3 7.5V6ZM11 6C9.89543 6 9 6.89543 9 8V9L13 7.5V6H11Z" fill="#000000"/>
</svg>`

onMounted(async () => {
    if (!novuInbox.value) {
        console.error("Novu inbox container element not found");
        return;
    }

    try {
        const novu = new NovuUI({
            options: {
                apiUrl: import.meta.env.VITE_APP_API_URL,
                socketUrl: import.meta.env.VITE_APP_SOCKET_URL,
                applicationIdentifier: import.meta.env.VITE_APP_APP_ID,
                subscriberId: import.meta.env.VITE_APP_SUBSCRIBER_ID
            }
        });

        const appearance = {
            variables: {
                borderRadius: '12px',
                fontSize: '24px',
                colorShadow: 'black',
                colorNeutral: 'tan',
                colorCounterForeground: 'teal',
                colorCounter: 'red',
                colorSecondaryForeground: 'orange',
                colorSecondary: 'green',
                colorPrimaryForeground: 'purple',
                colorPrimary: 'white',                
                colorForeground: 'blue',
                colorBackground: 'yellow'
            },
            elements: {
                notificationTextContainer: {
                    margin: '10px'
                },
                notificationSubject: {
                    fontWeight: 'bold'
                }
            }
        };

        const tabs = [
            {
                label: 'All notifications',
                filter: { tags: [] }
            },
            {
                label: 'Security Alerts',
                filter: { tags: ['alert'] }
            }
        ]

        novu.mountComponent({
            name: "Inbox",
            props: {
                // open: true,
                onNotificationClick,
                //renderNotification,
                appearance,
                tabs
            },
            element: novuInbox.value
        });

        novu.updateAppearance(appearance);
        novu.updateTabs(tabs);
        novuInstance = novu;
    }
    catch (error) {
        console.error("Failed to initialize Novu inbox", error);
    }
})

onUnmounted(() => {
    if (novuInstance && novuInbox.value) {
        try {
            novuInstance.unmountComponent(novuInbox.value);
        }
        catch (error) {
            console.error("Failed to unmount Novu inbox", error);
        }
    }
})


</script>