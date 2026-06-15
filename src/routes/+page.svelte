<script lang="ts">
	import { onMount } from 'svelte';
	import { io, type Socket } from 'socket.io-client';

	let socket: Socket | null = null;
	let connected = $state(false);

	let roomInput = $state('test-room');
	let currentRoom = $state<string | null>(null);

	let input = $state('');
	let messages= $state<string[]>([]);

	onMount(() => {
		socket = io('https://demochatback-production.up.railway.app');

		socket.on('connect', () => {
			connected = true;
			console.log('connected:', socket?.id);
		});

		socket.on('disconnect', () => {
			connected = false;
			currentRoom = null;
		});

		socket.on('joined-room', ({ roomId }) => {
			currentRoom = roomId;
			messages = [];
			console.log('joined room:', roomId);
		});

		socket.on('chat-message', (message: { body: string }) => {
			messages = [...messages, message.body];
		});

		return () => {
			socket?.disconnect();
		};
	});

	function joinRoom() {
		if (!socket || !roomInput.trim()) return;

		socket.emit('join-room', {
			roomId: roomInput.trim()
		});
	}

	function sendMessage() {
		if (!socket || !input.trim() || !currentRoom) return;

		socket.emit('chat-message', {
			roomId: currentRoom,
			body: input
		});

		input = '';
	}
</script>

<h1>Socket Playground</h1>

<p>Status: {connected ? 'connected' : 'disconnected'}</p>
<p>Current room: {currentRoom ?? 'none'}</p>

<div>
	<input bind:value={roomInput} placeholder="room id" />
	<button onclick={joinRoom}>Join room</button>
</div>

<div>
	<input
		bind:value={input}
		placeholder="message"
		disabled={!currentRoom}
		onkeydown={(e) => e.key === 'Enter' && sendMessage()}
	/>
	<button onclick={sendMessage} disabled={!currentRoom}>Send</button>
</div>

<ul>
	{#each messages as message}
		<li>{message}</li>
	{/each}
</ul>