<template>


    <div class="app-container">
        <!--        Toggle button for showing/hiding the chart-->
        <button @click.native="toggleChart" class="toggle-chart-btn">
            <span>Toggle Chart</span>
        </button>


        <!--        Content container for chat and optional chart-->
        <div class="content-container">
            <!-- Chat component -->
            <chat class="chat"
                  @workflowStarted="onWorkflowStarted"
                  @workflowUpdated="onWorkflowUpdated"
            />

            <!-- Chart component with v-show to toggle visibility -->
            <div class="chart-container">
                <vue-mermaid-string
                        v-if="showChart"
                        :value="mmFlow + mmFlowHighlight"/>
            </div>

        </div>
    </div>
</template>


<script>
import chat from "./components/Chat.vue"
import VueMermaidString from "vue-mermaid-string";

export default {
    components: {
        chat,
        VueMermaidString
    },

    data() {
        return {
            showChart: true,
            mmFlow: `  graph TD
    step1[Basic information setup] -->|Client submits name & email| step2[Application level encryption setup]
    step2 -->|Client selects key format and submits key/certificate| step3[API services setup]
    step3 -->|Client selects Payment Service| step4[Payment services setup]
    step3 -->|Client selects Account Information Service| step5[Account information services setup]
    step4 --> step6[Summary]
    step5 --> step6[Summary]
    step6 -->|Onboarding process summarized| End((Onboarding Complete))`,
            mmFlowHighlight: "\nstyle step1 fill:#f9f,stroke:#333,stroke-width:4px"
        };
    },
    mounted() {
        this.showChart = false;
    },

    methods: {
        toggleChart() {
            this.showChart = !this.showChart;
            console.log('Chart visibility:', this.showChart ? 'Visible' : 'Hidden');
        },

        onWorkflowStarted(workFlowStartedEvent) {
            console.log('workflow started: ' + workFlowStartedEvent.workflowName);

            this.showChart = true;
            this.mmFlow = workFlowStartedEvent.mmFlow;
            this.mmFlowHighlight = `\n\n     style ${workFlowStartedEvent.activeStep} fill:#f9f,stroke:#333,stroke-width:4px`;
        },

        onWorkflowUpdated(workflowUpdateEvent) {
            this.showChart = true;
            this.mmFlowHighlight = `\n\n     style ${workflowUpdateEvent.activeStep} fill:#f9f,stroke:#333,stroke-width:4px`;
        }

    }
    // ... your other component options ...
};
</script>

<style>
.app-container {
    position: relative;
}

.content-container {
    display: flex;
    height: 100vh;
}

.chat {
    flex-grow: 1;
}

.chart-container {
    /*border: 1px solid #000; !* 1px solid border in black *!*/
    /* Add other styling as needed */
}

.chart {
    width: 200px; /* Fixed width for the chart component */
    flex-shrink: 0; /* Prevent the chart from shrinking */
    display: none; /* Initially not displayed */
}

.toggle-chart-btn {
    position: absolute;
    top: 10px; /* Adjust as needed for spacing from the top edge */
    right: 10px; /* Adjust as needed for spacing from the right edge */
    z-index: 10;
    background-color: #f0f0f0;
    border: none;
    border-radius: 5px;
    padding: 10px 20px;
    cursor: pointer;
    font-size: 16px;
}

.toggle-chart-btn:hover {
    background-color: #e0e0e0;
}
</style>