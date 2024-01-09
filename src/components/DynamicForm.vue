<template>
    <form @submit.prevent="submitForm">
        <h2>{{ schema.title }}</h2>
        <div v-for="field in schema.fields" :key="field.model" class="form-group"
             v-show="isVisible(field)">

            <label :for="field.model">{{ field.label }}</label>
            <input
                    v-if="field.type === 'text' || field.type === 'email' || field.type === 'tel'"
                    :type="field.type"
                    :id="field.model"
                    v-model="formData[field.model]"
                    :placeholder="field.placeholder"
                    :disabled="isSubmitted"
                    class="form-control"
            />
            <!-- Error message for email -->
            <div v-if="field.type === 'email' && formData[field.model] && !isValidEmail(formData[field.model])"
                 class="error-message">
                Please enter a valid email address.
            </div>
            <!-- Error message for telephone -->
            <div v-if="field.type === 'tel' && formData[field.model] && !isValidPhone(formData[field.model])"
                 class="error-message">
                Please enter a valid telephone number.
            </div>
            <!--            single select -->
            <select
                    v-if="field.type === 'select'"
                    :id="field.model"
                    v-model="formData[field.model]"
                    :disabled="isSubmitted"
                    class="form-control"
            >
                <option v-for="option in field.options" :key="option.value" :value="option.value">
                    {{ option.text }}
                </option>
            </select>
            <!--            upload file -->
            <input
                    v-if="field.type === 'file'"
                    type="file"
                    :id="field.model"
                    @change="handleFileChange($event, field.model)"
                    :disabled="isSubmitted"
                    class="form-control"
            />
            <!-- Checkbox group for multiselect -->
            <div v-if="field.type === 'multiselect'" class="form-group">
                <div v-for="option in field.options" :key="option.value" class="form-check">
                    <input
                            type="checkbox"
                            :id="`${field.model}_${option.value}`"
                            :value="option.value"
                            :checked="formData[field.model][option.value]"
                            @change="handleCheckboxChange($event, field.model, option.value)"
                            class="form-check-input"
                            :disabled="isSubmitted"
                    />
                    <label :for="`${field.model}_${option.value}`" class="form-check-label">
                        {{ option.text }}
                    </label>
                </div>
            </div>
        </div>

        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
</template>

<script>
/* eslint-disable no-useless-escape */
export default {
    props: {
        schema: {
            type: Object,
            required: true
        }
    },
    data() {
        return {
            isSubmitted : false,
            formData: {}
        };
    },
    created() {
        this.initializeFormData();
    },
    methods: {
        initializeFormData() {
            this.schema.fields.forEach(field => {
                // Directly assign an initial value to the formData object
                this.formData[field.model] = field.type === 'file' ? [] : '';
            });

        },
        handleFileChange(event, model) {
            // Assuming single file for simplicity
            this.formData[model] = event.target.files[0];
        },
        isValidEmail(email) {
            const re = /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
            return re.test(email);
        },
        isValidPhone(phone) {
            const re = /^\+?\d{10,15}$/;
            return re.test(phone);
        },
        isVisible(field) {
            // If there's no visible_expr, the field is always visible
            if (!field.visible_expr) {
                return true;
            }
            // Create a new Function that returns the result of the expression
            // WARNING: Using Function constructor can pose security risks and should be avoided if possible
            try {
                const isVisible = new Function('formData', `return ${field.visible_expr}`);
                return isVisible(this.formData);
            } catch (e) {
                console.error('Error evaluating visible_expr:', e);
                return false; // If there is an error in the expression, hide the field by default
            }
        },
        handleCheckboxChange(event, model, optionValue) {
            // Set the value of the checkbox to the formData object
            // This ensures formData[model] is an object with keys as option.value
            if (!this.formData[model]) {
                this.formData[model] = {};
            }
            this.formData[model][optionValue] = event.target.checked;
        },
        submitForm() {
            // Handle form submission logic here
            this.isSubmitted = true;
            this.$emit('formSubmitted', {"formData": this.formData, "stepName": this.schema.title});
        }

    }
};
</script>

<style>
.form-group {
    margin-bottom: 1rem;
}

.form-control {
    width: 100%;
    padding: 0.75rem;
    margin-top: 0.5rem;
    border: 1px solid #ced4da;
    border-radius: 0.25rem;
}

.btn-primary {
    background-color: #007bff;
    border-color: #007bff;
    color: white;
    padding: 0.75rem 1.25rem;
    border-radius: 0.25rem;
    cursor: pointer;
}

.btn-primary:hover {
    background-color: #0069d9;
    border-color: #0062cc;
}

.error-message {
    color: red;
    font-size: 0.875em;
    margin-top: 0.25rem;
}
</style>