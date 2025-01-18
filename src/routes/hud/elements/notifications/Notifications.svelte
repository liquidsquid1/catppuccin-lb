<script lang="ts">
    import {flip} from "svelte/animate";
    import {listen} from "../../../../integration/ws";
    import {fly} from "svelte/transition";
    import Notification from "./Notification.svelte";
    import type {NotificationEvent} from "../../../../integration/events";
    import { Howl } from "howler";

    interface TNotification {
        animationKey: number;
        id: number;
        title: string;
        severity: string;
        message: string;
    }

    let notifications: TNotification[] = [];

    const notification = new Howl({
        src: ['audio/notifications/notification.mp3'],
        preload: true
    });

    const error = new Howl({
        src: ['audio/notifications/error.mp3'],
        preload: true
    });

    const info = new Howl({
        src: ['audio/notifications/info.mp3'],
        preload: true
    });

    const success = new Howl({
        src: ['audio/notifications/success.ogg'],
        preload: true
    });

    function addNotification(title: string, message: string, severity: string) {
        let animationKey = Date.now();
        const id = animationKey;

        if (severity === "ENABLED" || severity === "DISABLED") {
            const index = notifications.findIndex((n) => n.message === message)
            if (index !== -1) {
                animationKey = notifications[index].animationKey;

                notifications.splice(index, 1);
            }
        }

        notifications = [
            {animationKey, id, title, message, severity},
            ...notifications,
        ];
        
        setTimeout(() => {
            notifications = notifications.filter((n) => n.id !== id);
        }, 3000);
    }

    listen("notification", (e: NotificationEvent) => {
        addNotification(e.title, e.message, e.severity);

        if (e.severity === "ERROR") {
            error.play();
        } else
        if (e.severity === "INFO") {
            info.play();
        } else
        if (e.severity === "SUCCESS") {
            success.play();
        } else {
            notification.play();
        }

    });

    addNotification("Catppuccin", "Thank you for using my theme <3", "INFO")
</script>

<div class="notifications">
    {#each notifications as {title, message, severity, animationKey} (animationKey)}
        <div
                animate:flip={{ duration: 200 }}
                in:fly={{ x: 30, duration: 200 }}
                out:fly={{ x: 30, duration: 200 }}
        >
            <Notification {title} {message} {severity}/>
        </div>
    {/each}
</div>
