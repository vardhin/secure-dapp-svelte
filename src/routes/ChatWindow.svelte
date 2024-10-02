<script>
    import { page } from '$app/stores';
    import { onMount } from 'svelte';
    import {
        getContactHistory,
        getContactDetails,
        addMessageToContactHistory
    } from '../lib/utils/contactManager';
    import { sendMessageToDB, listenToUpdates } from '../lib/utils/firearms';
    import { darkMode } from '../lib/utils/stores'; // Import dark mode store
    import Navbar from './Navbar.svelte'; // Import the Navbar component

    let contactName = '';
    let history = [];
    let message = '';
    let isLoading = true;
    let contactDetails = null;
    let stopListening;
    let isDarkMode = false;

    $: {
        const { state } = $page;
        contactName = state?.contactName || 'Unknown Contact';
    }

    // Subscribe to darkMode store
    darkMode.subscribe((value) => {
        isDarkMode = value;
    });

    const fetchData = async () => {
        try {
            console.log('Fetching data for contact:', contactName);
            isLoading = true;

            // Get contact details
            contactDetails = await getContactDetails(contactName);
            console.log('Contact details fetched:', contactDetails);

            // Get contact history
            history = await getContactHistory(contactName);
            console.log('Contact history fetched:', history);

            isLoading = false;

            // Start listening to updates
            if (contactDetails && contactDetails.id && contactDetails.name) {
                stopListening = listenToUpdates(
                    contactDetails.id, // Current user ID
                    contactDetails.name, // Friend name
                    (newHistory) => {
                        console.log('New history received:', newHistory);
                        history = [...history, ...newHistory]; // Updates chat history in real-time
                    }
                );
            }
        } catch (error) {
            console.error('Error fetching data: ', error);
            isLoading = false;
        }
    };

    onMount(() => {
        if (contactName) {
            fetchData();
        }
        return () => {
            if (stopListening) stopListening();
        };
    });

    const handleSendMessage = async (e) => {
        e.preventDefault();
        try {
            if (!message.trim()) {
                alert('Message cannot be empty!');
                return;
            }

            console.log('Sending message:', message);
            await sendMessageToDB(contactDetails.id, contactDetails.friendId, message);
            await addMessageToContactHistory(contactName, {
                message,
                timestamp: Date.now()
            });

            message = ''; // Clear the input after sending
        } catch (error) {
            console.error('Error sending message: ', error);
        }
    };
</script>

<body>
    <!-- Use Navbar and pass the title dynamically -->
    <Navbar title={contactName} />

    <main class:dark-mode={isDarkMode}>
        {#if isLoading}
            <p>Loading chat history...</p>
        {:else}
            <div>
                {#if contactDetails}
                    <h2>Contact Details</h2>
                    <p>Name: {contactDetails.name}</p>
                    <p>ID: {contactDetails.id}</p>
                    <p>Friend ID: {contactDetails.friendId}</p>
                {/if}

                {#if history.length > 0}
                    <h3>Contact History</h3>
                    <ul>
                        {#each history as entry, index}
                            {#if typeof entry === 'object' && entry.message && entry.timestamp && entry.id}
                                <li key={entry.id || index}>
                                    <p>
                                        {entry.id === contactDetails.id ? 'Me' : contactDetails.name}
                                        : {entry.message} (Sent at: {new Date(entry.timestamp).toLocaleString()})
                                    </p>
                                </li>
                            {/if}
                        {/each}
                    </ul>
                {/if}

                <form on:submit={handleSendMessage}>
                    <input type="text" bind:value={message} placeholder="Type your message" required />
                    <button type="submit">Send</button>
                </form>
            </div>
        {/if}
    </main>
</body>

<style>
    @import '@fontsource/orbitron/400.css';
    @import '@fontsource/orbitron/700.css';

    :root {
        --bg-color: #f5f5f5;
        --form-bg-color: #fff;
        --text-color: #333;
        --button-bg-color: #4caf50;
        --button-hover-bg-color: #45a049;
        --body-bg-color: #2f2243;
        --body-text-color: #333;
    }

    .dark-mode {
        --bg-color: #1a1a1a;
        --form-bg-color: #222;
        --text-color: #ddd;
        --button-bg-color: #00e0ff;
        --button-hover-bg-color: #00ff99;
        --body-bg-color: #444;
        --body-text-color: #ddd;
    }

    main {
        background-color: var(--body-bg-color);
        color: var(--body-text-color);
        font-family: 'Orbitron', sans-serif;
        padding: 20px;
        height: 100vh;
        transition:
            background-color 0.3s ease,
            color 0.3s ease;
    }

    body {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
        height: 100%;
        width: 100%;
    }

    h1 {
        color: var(--text-color);
        font-size: 2rem;
        margin-bottom: 20px;
        transition: color 0.3s ease;
        text-align: center;
    }

    h2,
    h3 {
        color: var(--text-color);
        font-size: 1.5rem;
        transition: color 0.3s ease;
    }

    ul {
        list-style-type: none;
        padding: 0;
        margin: 20px 0;
    }

    li {
        background-color: var(--form-bg-color);
        border-radius: 8px;
        padding: 10px;
        margin-bottom: 10px;
        transition: background-color 0.3s ease;
    }

    p {
        color: var(--text-color);
        font-size: 1.1rem;
    }

    form {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-top: 20px;
    }

    input[type='text'] {
        flex: 1;
        padding: 10px;
        border: none;
        border-radius: 8px;
        font-size: 1.1rem;
        background-color: var(--form-bg-color);
        color: var(--text-color);
        transition:
            background-color 0.3s ease,
            color 0.3s ease;
        margin-right: 10px;
    }

    input[type='text']::placeholder {
        color: var(--text-color);
    }

    button {
        background-color: var(--button-bg-color);
        color: #fff;
        border: none;
        border-radius: 8px;
        padding: 10px 20px;
        font-size: 1.1rem;
        cursor: pointer;
        transition:
            background-color 0.3s ease,
            transform 0.2s ease;
    }

    button:hover {
        background-color: var (--button-hover-bg-color);
        transform: translateY(-2px);
    }

    button:active {
        transform: translateY(1px);
    }
</style>
