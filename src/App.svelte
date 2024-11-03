<script>
	import Button from '@smui/button';
	import TextField from '@smui/textfield';
	import Paper from '@smui/paper';
	import IconButton from '@smui/icon-button';
  
	let showChatbot = false;
	let messages = [];
	let userMessage = "";
  
	function toggleChatbot() {
	  showChatbot = !showChatbot;
	}

	async function handleSubmit() {
		const newMessage = {
			role: 'user',
			content: userMessage.trim()
		};
		const response = await fetch(
        `https://dev11-app.csnonprod.com/automations-api/run/agents/82652150c81940d5bc5af534be84149e`,
        {
          method: 'POST',
          headers: {
			'Content-Type': 'application/json',
			'authtoken': 'bltdfccda52aa5387d1',
			'organization_uid': 'blt87b0a3aff3fc7a51',
		  },
          body: JSON.stringify({
            messages: [...messages, newMessage],
            stream: true
          })
        }
      );

      if (!response.body) {
        throw new Error('No response body');
      }

      const reader = response.body
        .pipeThrough(new TextDecoderStream())
        .getReader();

      while (true) {
        const { value, done } = await reader.read();
		console.log('Value:', value);
        if (done) break;
	}
	}
  
	async function sendMessage() {
	  await handleSubmit();
	  if (userMessage.trim()) {
		messages = [...messages, { role: "user", text: userMessage }];
		setTimeout(() => {
		  messages = [...messages, { role: "bot", text: "I'm here to help!" }];
		}, 1000);
		userMessage = "";
	  }
	}
  </script>
  
  <style>
	.floating-button {
	  position: fixed;
	  bottom: 20px;
	  right: 20px;
	  background-color: #4CAF50;
	  color: white;
	  border-radius: 50%;
	  width: 60px;
	  height: 60px;
	  display: flex;
	  align-items: center;
	  justify-content: center;
	  cursor: pointer;
	  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.3);
	  z-index: 1000;
	}
  
	.chat-container {
	  position: fixed;
	  bottom: 20px;
	  right: 20px;
	  width: 60px;
	  height: 60px;
	  overflow: hidden;
	  border-radius: 50%;
	  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.3);
	  transition: width 0.3s ease, height 0.3s ease, border-radius 0.3s ease;
	}
  
	/* Expanded Chat Window */
	.chat-container.expanded {
	  width: 400px;
	  height: 500px;
	  border-radius: 10px;
	}
  
	.chat-window {
	  display: flex;
	  flex-direction: column;
	  height: 100%;
	  width: 100%;
	}
  
	.messages {
	  flex: 1;
	  overflow-y: auto;
	  padding: 1rem;
	  background-color: #f5f5f5;
	}
  
	.message {
	  margin-bottom: 1rem;
	}
  
	.bot-message {
	  background-color: #e0e0e0;
	  padding: 0.5rem 1rem;
	  border-radius: 8px;
	  align-self: flex-start;
	}
  
	.user-message {
	  background-color: #4CAF50;
	  color: white;
	  padding: 0.5rem 1rem;
	  border-radius: 8px;
	  align-self: flex-end;
	}
  
	.input-area {
	  display: flex;
	  padding: 1rem;
	  background-color: #fff;
	  border-top: 1px solid #ddd;
	}
  </style>
  
  <!-- Floating Button to Toggle Chatbot -->
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <div class="floating-button" on:click={toggleChatbot}>
	<IconButton class="material-icons"></IconButton>
  </div>
  
  <!-- Chatbot Container -->
  <div class="chat-container {showChatbot ? 'expanded' : ''}">
	{#if showChatbot}
	  <Paper class="chat-window" elevation={3}>
		<div class="messages">
		  {#each messages as message}
			<div class="message">
			  {#if message.sender === "bot"}
				<div class="bot-message">{message.content}</div>
			  {:else}
				<div class="user-message">{message.content}</div>
			  {/if}
			</div>
		  {/each}
		</div>
  
		<div class="input-area">
		  <TextField 
			bind:value={userMessage} 
			label="Type your message..." 
			variant="outlined" 
			fullWidth 
			on:keyup={(e) => e.key === 'Enter' && sendMessage()}
		  />
		  <Button variant="contained" color="primary" on:click={sendMessage}>Send</Button>
		</div>
	  </Paper>
	{/if}
  </div>
  