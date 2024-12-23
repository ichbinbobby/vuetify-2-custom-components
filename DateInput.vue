<template>
    <v-menu
        v-model="menu"
        :close-on-content-click="false"
        :nudge-right="40"
        transition="scale-transition"
        offset-y
        min-width="auto"
    >
        <template v-slot:activator="{ on, attrs }">
            <!-- Will take the available width -->
            <v-text-field
                v-model="text"
                clearable
                hide-details="auto"
                placeholder="dd.MM.yyyy"
                :dense="dense"
                :label="label"
                :rules="[rules.date]"
                @click:prepend="on.click"
                @click:clear="clear"
            >
                <template v-slot:prepend>
                    <v-icon
                        v-bind="attrs"
                        v-on="on"
                        @click="menu = true"
                    >
                        {{ mdiCalendar }}
                    </v-icon>
                </template>
            </v-text-field>
        </template>

        <v-date-picker
            v-model="date"
            scrollable
            :locale="$datepickerLocale()"
            @input="menu = false"
        ></v-date-picker>
    </v-menu>
</template>

<script>
import { mdiCalendar } from '@mdi/js';

export default {
    props: {
        dense: {
            type: Boolean,
            default: true
        },

        label: {
            type: String,
            default: ''
        },

        // Accept timestamp or string in German format
        value: {
            type: [Number, String],
            default: '',
            validator: value => {
                return typeof value === 'number' || (typeof value === 'string' && /^\d{2}\.\d{2}\.\d{4}$/.test(value));
            }
        }
    },

    data() {
        return {
            date: null,
            menu: false,
            // text === internalValue
            text: this.value,

            // German date format
            regex: /^\d{2}\.\d{2}\.\d{4}$/,
            rules: {
                date: value => {
                    return this.regex.test(value) || this.$t('txt.rules.dateFormat');
                }
            },

            mdiCalendar: mdiCalendar
        };
    },

    mounted() {
        this.updateInternalValues(this.value);
    },

    /*
     * date watcher has to set the value of text and vice versa
     * both have to set the internalValue
     */
    watch: {
        date(newValue) {
            // Textfield should display date in German format
            this.text = new Date(newValue).toLocaleDateString('de-DE', {
                day: '2-digit',
                month: '2-digit',
                year: 'numeric'
            });

            this.emitValue();
        },

        text(newValue) {
            // If user typed the full date
            if (newValue && newValue.length === 10 && this.regex.test(newValue)) {
                // Split the German date string by the dot (.) separator
                const [day, month, year] = newValue.split('.');

                this.date = `${year}-${month}-${day}`; // ISO format YYYY-MM-DD

                this.emitValue();
            }
        },

        value(newValue) {
            this.updateInternalValues(newValue);
        }
    },

    methods: {
        clear() {
            this.date = null;
            this.text = '';
            this.emitValue();
        },

        emitValue() {
            if (typeof this.value === 'number') {
                this.$emit('input', new Date(this.date).getTime());
            } else {
                this.$emit('input', this.text);
            }
        },

        updateInternalValues(value) {
            if (typeof value === 'number') {
                this.date = new Date(value).toLocaleDateString('en-CA');
                this.text = new Date(value).toLocaleDateString('de-DE', {
                    day: '2-digit',
                    month: '2-digit',
                    year: 'numeric'
                });
            } else if (typeof value === 'string' && value !== '') {
                const [day, month, year] = value.split('.');
                this.date = `${year}-${month}-${day}`;
                this.text = value;
            }
        }
    }
};
</script>
