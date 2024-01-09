<!--suppress ALL -->
<template>
    <div class="chat-container">
        <!-- Context menu trigger and items container -->
        <div class="context-menu-container">
            <!-- Context menu icon -->
            <div class="context-menu-icon" @click="toggleContextMenu">
                ☰
                <!-- This is a simple representation of the menu icon; you might want to use an SVG or an icon library -->
            </div>
            <!-- Hidden context menu items -->
            <div v-if="showContextMenu" class="menu-items">
                <div class="menu-item" @click="clearHistory">
                    Clear History <!-- You can replace this with an SVG or an icon from a library -->
                </div>
                <div class="menu-item" @click="attachFile">
                    Attach File <!-- You can replace this with an SVG or an icon from a library -->
                    <input type="file" ref="fileInput" @change="uploadPDF" accept=".pdf" hidden/>
                </div>
            </div>
        </div>

        <div class="messages" ref="messagesContainer">
            <div v-for="(message, index) in messageHistory"
                 :key="index"
                 class="message"
                 :class="{ 'user': message.sender === 'user', 'system': message.sender === 'system' }">
                <pre v-if="message.sender === 'user'"
                     class="message-text">{{ message.text }}</pre>
                <pre v-if="message.sender === 'system' && message.type === 'message'"
                     class="message-text">{{ message.text }}</pre>
                <DynamicForm v-if="message.sender === 'system' && message.type === 'form-schema'"
                             :schema="message.text.form_schema"
                             @formSubmitted="onFormSubmitted"
                             class="my-form"/>
            </div>
        </div>
        <div class="input-area">
            <textarea v-model="newMessage"
                      @keydown="handleKeydown"
                      placeholder="Type a message..."/>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import DynamicForm from './DynamicForm.vue';


export default {
    // eslint-disable-next-line vue/multi-word-component-names
    name: 'Chat',
    components: {
        DynamicForm
    },
    data() {
        return {
            newMessage: '',
            messageHistory: [],
            showContextMenu: false
        };
    },
    mounted() {
        this._mounted();
    },
    methods: {
        _mounted() {
            // Access the DOM element via this.$refs and call focus()
            this.addToChatHistory({
                text: "I am a bank assistant, the only task I can do is to onboard client to bank's API service, do want to begin the onboarding now ?",
                sender: 'system',
                type: "message"
            });
        },

        handleKeydown(event) {
            if (event.key === 'Enter') {
                if (event.shiftKey) {
                    // Shift + Enter was pressed; insert new line
                    // Default behavior of Enter in a textarea is to add a new line,
                    // so we don't need to do anything here.
                } else {
                    // Just Enter was pressed; send message
                    event.preventDefault(); // Prevent default to avoid new line in textarea
                    this.sendMessage();
                }
            }
        },

        addNewLine(event) {
            // Insert a new line at the current cursor position
            const cursorPosition = event.target.selectionStart;
            const textBeforeCursor = this.newMessage.slice(0, cursorPosition);
            const textAfterCursor = this.newMessage.slice(cursorPosition);

            this.newMessage = textBeforeCursor + "\n" + textAfterCursor;
            // Move the cursor to the next line
            this.$nextTick(() => {
                event.target.selectionStart = cursorPosition + 1;
                event.target.selectionEnd = cursorPosition + 1;
            });
        },

        // send the user input message to backend
        async sendMessage() {
            if (this.newMessage.trim() === '') return;

            const userMessage = this.newMessage;
            this.addToChatHistory({text: userMessage, sender: 'user'});
            this.newMessage = '';

            try {
                const httpResponse = await axios.post(
                    'https://c8447254-4de9-4f37-8fe2-53acec60c1f8.mock.pstmn.io/user-messages?message-type=first-message',
                    {
                        "user_message": "yes"
                    });
                const systemResponse = httpResponse.data;

                console.log("system response: ");
                console.dir(systemResponse);

                if (systemResponse.workflow_status === "newborn") {
                    this.$emit('workflowStarted', {
                        workflowName: systemResponse.workflow_name,
                        mmFlow: systemResponse.mm_flow,
                        activeStep: systemResponse.active_step

                    });
                }
                this.addToChatHistory({text: systemResponse, sender: 'system', type: "form-schema"});
            } catch (error) {
                console.error('API call failed:', error);
                this.addToChatHistory({text: 'Failed to get response from the system.', sender: 'system'});
            }
        },
        addToChatHistory(message) {
            this.messageHistory.push(message);
            this.$nextTick(() => {
                this.scrollToBottom();
            });
        },

        scrollToBottom() {
            const messagesContainer = this.$refs.messagesContainer;
            messagesContainer.scrollTop = messagesContainer.scrollHeight;
        },

        toggleContextMenu() {
            this.showContextMenu = !this.showContextMenu;
        },
        clearHistory() {
            this.messageHistory = []; // Assuming 'messages' is the data property holding the chat history
            this.showContextMenu = false;
            this._mounted();
        },
        attachFile() {
            this.$refs.fileInput.click();
            this.showContextMenu = false;
        },
        uploadPDF(event) {
            const file = event.target.files[0];
            if (file) {
                // TODO: Implement file uploading logic here
                console.log('File selected for upload:', file.name);
            }
        },

        async onFormSubmitted(submitEvent) {
            // Handle the submitted form value here
            console.log('Form submitted with value:', submitEvent);

            const httpResponse = await axios.post(
                'https://c8447254-4de9-4f37-8fe2-53acec60c1f8.mock.pstmn.io/form-submissions?finishing-step=' + submitEvent.stepName,
                submitEvent.formData);

            const systemResponse = httpResponse.data;
            console.log("system response: ");
            console.dir(systemResponse);

            if (systemResponse.workflow_status === "live") {
                this.$emit('workflowUpdated', {
                    workflowName: systemResponse.workflow_name,
                    activeStep: systemResponse.active_step
                });
            }

            this.addToChatHistory({text: systemResponse, sender: 'system', type: "form-schema"});
        }
    }
}
</script>

<style>
.chat-container {
    display: flex;
    flex-direction: column;
    max-height: 685px;
}

.messages {
    flex-grow: 1;
    overflow-y: auto;
}

.message {
    margin-bottom: 10px;
    padding: 5px;
    border-radius: 5px;
}

.message .message-text {
    white-space: pre-wrap; /* CSS property that will maintain whitespace and line breaks */
    word-wrap: break-word; /* To prevent long text strings from overflowing */
    /* ... additional styling ... */
}

.message.user {
    align-self: flex-end;
    background-color: #d1ecf1;
}

.message.system {
    align-self: flex-start;
    background-color: whitesmoke;
}

.input-area {
    display: flex;
    margin-top: 10px;
}

.input-area input {
    flex-grow: 1;
    margin-right: 10px;
}

.input-area button {
    padding: 5px 10px;
}

.input-area textarea {
    width: 100%; /* Set width to 100% of the parent */
    box-sizing: border-box; /* Include padding and border in the element's width and height */
    padding: 5px; /* Adjust padding as needed */
    margin-right: 0; /* Remove margin if it was previously set */
    border: 1px solid #ccc; /* Optional: set a border for the textarea */
    border-radius: 4px; /* Optional: set a border-radius for the textarea */
    resize: vertical; /* Allow only vertical resizing */
}

.context-menu-container {
    position: relative;
    display: inline-block; /* or flex depending on your layout */
    /* Additional styling */
}

.context-menu-icon {
    cursor: pointer;
    /* Style the menu icon as needed */
    /* Ensure it has some margin if you want to separate it from the menu */
}

.menu-items {
    display: block;
    /*left: 100px;*/
    top: 100%; /* Position the menu items directly below the menu icon */
    background-color: #f9f9f9;
    min-width: 150px;
    box-shadow: 0px 8px 16px 0px rgba(0, 0, 0, 0.2);
    z-index: 1;
    /* Additional styles for the menu */
}

.menu-item {
    padding: 12px 16px;
    cursor: pointer;
    /* Styles for each menu item */
}

.menu-item:hover {
    background-color: #f1f1f1; /* Hover effect for menu items */
}

.my-form {
    width: calc(100% - 2em);
    max-width: 480px;
    box-sizing: border-box;
    padding: 2em;
    box-shadow: 0 0 1em rgba(0, 0, 0, .1);
    border-radius: .5em;
    margin: 4em auto;
}
</style>