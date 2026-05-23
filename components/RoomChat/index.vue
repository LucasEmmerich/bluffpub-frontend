<template>
    <div
        class="flex flex-col rounded-xl overflow-hidden"
        style="
            width: 260px;
            height: 180px;
            background: rgba(8, 5, 3, 0.92);
            border: 1px solid rgba(255, 255, 255, 0.18);
            backdrop-filter: blur(10px);
            box-shadow: 0 4px 32px rgba(0, 0, 0, 0.7);
        "
    >
        <div class="shrink-0 px-3 py-1.5" style="border-bottom: 1px solid rgba(255, 255, 255, 0.08)">
            <span class="font-pub text-[9px] tracking-[0.35em] uppercase" style="color: rgba(255, 248, 238, 0.45)"
                >Room Chat</span
            >
        </div>

        <div ref="msgList" class="flex-1 overflow-y-auto chat-scroll flex flex-col gap-px px-2 py-2" style="min-height: 0">
            <div
                v-if="roomMessages.length === 0"
                class="flex-1 flex items-center justify-center font-pub text-[9px] tracking-wide"
                style="color: rgba(255, 248, 238, 0.15)"
            >
                No messages yet...
            </div>
            <div
                v-for="msg in roomMessages"
                :key="msg.id"
                class="flex items-start gap-1.5 py-1 px-1 rounded"
                :style="{ background: msg.playerId === mainPlayerId ? 'rgba(255,248,238,0.03)' : 'transparent' }"
            >
                <img
                    :src="avatars[msg.avatar]"
                    class="shrink-0 rounded-full object-cover"
                    style="width: 18px; height: 18px; margin-top: 1px"
                />
                <div class="flex flex-col min-w-0">
                    <span
                        class="font-pub text-[8px] tracking-wide leading-tight truncate"
                        :style="{
                            color: msg.playerId === mainPlayerId ? 'rgba(184,134,11,0.8)' : 'rgba(255,248,238,0.45)',
                        }"
                    >
                        {{ msg.username }}
                    </span>
                    <span
                        class="font-pub text-[10px] leading-snug break-words"
                        style="color: rgba(255, 248, 238, 0.75); word-break: break-word"
                    >
                        {{ msg.text }}
                    </span>
                </div>
            </div>
        </div>

        <div class="shrink-0 px-2 pb-2 pt-1" style="border-top: 1px solid rgba(255, 255, 255, 0.06)">
            <div
                class="flex gap-1.5 items-center rounded-lg px-2 py-1.5"
                style="background: rgba(255, 255, 255, 0.04); border: 1px solid rgba(255, 255, 255, 0.08)"
            >
                <input
                    v-model="draft"
                    @keydown.enter.prevent="send"
                    maxlength="200"
                    placeholder="Type a message..."
                    class="flex-1 bg-transparent outline-none font-pub text-[10px] min-w-0"
                    style="color: rgba(255, 248, 238, 0.8); caret-color: rgba(184, 134, 11, 0.8)"
                />
                <button
                    @click="send"
                    :disabled="!draft.trim()"
                    class="shrink-0 flex items-center justify-center rounded transition-opacity duration-150"
                    :style="{ opacity: draft.trim() ? '1' : '0.3' }"
                    style="
                        width: 20px;
                        height: 20px;
                        background: rgba(184, 134, 11, 0.2);
                        border: 1px solid rgba(184, 134, 11, 0.3);
                    "
                >
                    <svg
                        width="9"
                        height="9"
                        viewBox="0 0 24 24"
                        fill="none"
                        stroke="rgba(184,134,11,0.9)"
                        stroke-width="2.5"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                    >
                        <line x1="12" y1="19" x2="12" y2="5" />
                        <polyline points="5 12 12 5 19 12" />
                    </svg>
                </button>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { roomMessages, sendRoomMessage } from "~/composables/useChat";
import { avatars } from "~/assets/avatars";

const props = defineProps<{ roomId: string; mainPlayerId?: string }>();

const draft = ref("");
const msgList = ref<HTMLElement | null>(null);

const send = () => {
    const text = draft.value.trim();
    if (!text) return;
    const _room = useRoom();
    sendRoomMessage(props.roomId, _room.mainPlayer, text);
    draft.value = "";
};

watch(
    () => roomMessages.length,
    async () => {
        await nextTick();
        if (msgList.value) msgList.value.scrollTop = msgList.value.scrollHeight;
    }
);
</script>

<style scoped>
input::placeholder {
    color: rgba(255, 248, 238, 0.2);
}

.chat-scroll::-webkit-scrollbar {
    width: 3px;
}
.chat-scroll::-webkit-scrollbar-track {
    background: transparent;
}
.chat-scroll::-webkit-scrollbar-thumb {
    background: rgba(255, 255, 255, 0.08);
    border-radius: 2px;
}
</style>
