<template>
  <fieldset class="agenda-item-form">
    <button type="button" class="agenda-item-form__remove-button" @click="$emit('remove')">
      <ui-icon icon="trash" />
    </button>

    <ui-form-group>
      <ui-dropdown v-model="localItem.type" title="Тип" :options="$options.agendaItemTypeOptions" name="type" />
    </ui-form-group>

    <div class="agenda-item-form__row">
      <div class="agenda-item-form__col">
        <ui-form-group label="Начало">
          <ui-input
            v-model="localItem.startsAt"
            type="time"
            placeholder="00:00"
            name="startsAt"
            @change="changedStartTime"
          />
        </ui-form-group>
      </div>
      <div class="agenda-item-form__col">
        <ui-form-group label="Окончание">
          <ui-input v-model="localItem.endsAt" type="time" placeholder="00:00" name="endsAt" @change="changedEndTime"/>
        </ui-form-group>
      </div>
    </div>

    <ui-form-group v-for="(item, key) in $options.agendaItemFormSchemas[localItem.type]" :key="key" :label="item.label">
      <component :is="item.component" v-bind="item.props" v-model="localItem[key]"></component>
    </ui-form-group>
  </fieldset>
</template>

<script lang="ts">
import UiIcon from './UiIcon.vue';
import UiFormGroup from './UiFormGroup.vue';
import UiInput from './UiInput.vue';
import UiDropdown from './UiDropdown.vue';
import { defineComponent } from 'vue';
import type { PropType } from 'vue';

const agendaItemTypeIcons = {
  registration: 'key',
  opening: 'cal-sm',
  talk: 'tv',
  break: 'clock',
  coffee: 'coffee',
  closing: 'key',
  afterparty: 'cal-sm',
  other: 'cal-sm',
};

const agendaItemDefaultTitles = {
  registration: 'Регистрация',
  opening: 'Открытие',
  break: 'Перерыв',
  coffee: 'Coffee Break',
  closing: 'Закрытие',
  afterparty: 'Afterparty',
  talk: 'Доклад',
  other: 'Другое',
};

// eslint-disable-next-line @typescript-eslint/ban-ts-comment
// @ts-ignore
const agendaItemTypeOptions = Object.entries(agendaItemDefaultTitles).map(([type, title]) => ({
  value: type,
  text: title,
  // eslint-disable-next-line @typescript-eslint/ban-ts-comment
  // @ts-ignore
  icon: agendaItemTypeIcons[type],
}));

const talkLanguageOptions = [
  { value: null, text: 'Не указано' },
  { value: 'RU', text: 'RU' },
  { value: 'EN', text: 'EN' },
];

/**
 * @typedef FormItemSchema
 * @property {string} label
 * @property {string|object} component
 * @property {object} props
 */
/** @typedef {string} AgendaItemField */
/** @typedef {string} AgendaItemType */
/** @typedef {Object.<AgendaItemType, FormItemSchema>} FormSchema */

/** @type FormSchema */
const commonAgendaItemFormSchema = {
  title: {
    label: 'Нестандартный текст (необязательно)',
    component: 'ui-input',
    props: {
      name: 'title',
    },
  },
};

/** @type {Object.<AgendaItemField, FormSchema>} */
const agendaItemFormSchemas = {
  registration: commonAgendaItemFormSchema,
  opening: commonAgendaItemFormSchema,
  talk: {
    title: {
      label: 'Тема',
      component: 'ui-input',
      props: {
        name: 'title',
      },
    },
    speaker: {
      label: 'Докладчик',
      component: 'ui-input',
      props: {
        name: 'speaker',
      },
    },
    description: {
      label: 'Описание',
      component: 'ui-input',
      props: {
        multiline: true,
        name: 'description',
      },
    },
    language: {
      label: 'Язык',
      component: 'ui-dropdown',
      props: {
        options: talkLanguageOptions,
        title: 'Язык',
        name: 'language',
      },
    },
  },
  break: commonAgendaItemFormSchema,
  coffee: commonAgendaItemFormSchema,
  closing: commonAgendaItemFormSchema,
  afterparty: commonAgendaItemFormSchema,
  other: {
    title: {
      label: 'Заголовок',
      component: 'ui-input',
      props: {
        name: 'title',
      },
    },
    description: {
      label: 'Описание',
      component: 'ui-input',
      props: {
        multiline: true,
        name: 'description',
      },
    },
  },
};

interface DataSet {
  localItem: AgendaItem;
  startTime: number;
  endTime: number;
  eventDurationMils: number;
}

enum AgendaTypes {
  registration = 'registration',
  opening = 'opening',
  talk = 'talk',
  break = 'break',
  coffee = 'coffee',
  closing = 'closing',
  afterparty = 'afterparty',
  other = 'other',
}

interface AgendaItem {
  id: number;
  type: AgendaTypes;
  startsAt: string;
  endsAt: string;
  [key: string]: unknown;
}

export default defineComponent({
  name: 'MeetupAgendaItemForm',
  components: { UiIcon, UiFormGroup, UiInput, UiDropdown },

  agendaItemTypeOptions,
  agendaItemFormSchemas,

  props: {
    agendaItem: {
      type: Object as PropType<AgendaItem>,
      required: true,
    },
  },
  emits: ['remove', 'update:agendaItem'],
  data(): DataSet {
    const start = this.getTimeMils(this.agendaItem.startsAt);
    const end = this.getTimeMils(this.agendaItem.endsAt);
    return {
      localItem: { ...this.agendaItem },
      startTime: start,
      endTime: end,
      eventDurationMils: end - start,
    };
  },
  watch: {
    localItem: {
      deep: true,
      handler(newValue, oldValue): void {
        this.$emit('update:agendaItem', { ...newValue });
      },
    },
  },
  methods: {
    getTimeMils(time: string): number {
      const offset: number = new Date().getTimezoneOffset();
      const [hours, minutes]: number[] = time.split(':').map((val) => parseInt(val));
      return new Date(1970, 0, 1, hours, minutes).setMinutes(-1 * offset);
    },
    changedStartTime(event: Event): void {
      this.startTime = (event.target as HTMLInputElement).valueAsNumber;
      this.endTime = this.startTime + this.eventDurationMils;
      this.localItem.endsAt = new Date(this.endTime).toISOString().slice(11, 16);
    },
    changedEndTime(event: Event): void {
      this.endTime = (event.target as HTMLInputElement).valueAsNumber;
      this.eventDurationMils = this.endTime - this.startTime;
    },
  },
});
</script>

<style scoped>
.agenda-item-form {
  border: 2px solid var(--blue-light);
  border-radius: 8px;
  position: relative;
  padding: 20px 10% 0 16px;
}

.agenda-item-form__remove-button {
  position: absolute;
  top: 4px;
  right: 0;
  box-shadow: none;
  border: none;
  background-color: transparent;
  outline: none;
  padding: 4px;
  cursor: pointer;
  transition: 0.2s opacity;
}

.agenda-item-form__remove-button:hover {
  opacity: 0.6;
}

.agenda-item-form__row {
  display: flex;
  flex-direction: column;
}

.agenda-item-form__col + .agenda-item-form__col {
  margin-top: 16px;
}

.agenda-item-form__col:first-child {
  margin-left: 0;
}

@media all and (min-width: 992px) {
  .agenda-item-form {
    padding: 28px 25% 4px 24px;
  }

  .agenda-item-form__remove-button {
    top: 20px;
    right: 20px;
  }

  .agenda-item-form__row {
    flex-direction: row;
    justify-content: space-between;
    margin: 0 -12px;
  }

  .agenda-item-form__col {
    flex: 1 1 auto;
    padding: 0 12px;
  }

  .agenda-item-form__col + .agenda-item-form__col {
    margin-top: 0;
  }

  .agenda-item-form__col:first-child {
    margin-left: 0;
  }
}
</style>
