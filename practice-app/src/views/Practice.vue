<template>
  <div class="container">
    <h1>Practice</h1>

    <div class="row">
      <div class="practice-form">
        <label for="date" class="form-label">Date</label><br>
        <datepicker id="date" v-model="newPracticeItem.date" :disabled-dates="disabledDates"
                    :format="customFormatter" class="form-control"></datepicker>

        <label for="duration" class="form-label">Duration (min)</label><br>
        <input id="duration" v-model="newPracticeItem.duration" class="form-control" type="text"/>

        <label for="notes" class="form-label">Notes</label><br>
        <textarea id="notes" v-model="newPracticeItem.notes"
                  style="width: 400px" class="form-control"></textarea>
      </div>

    </div>
    <br>
    <button @click="addPractice()" class="btn btn-primary">Add</button>

    <br><br>
    <template v-if="practices.length > 0">
      <div class="container">
        <h2 style="text-align:left">My Practice Log</h2>
        <ul class="list-group list-group-flush">
          <li v-for="practice in practices" :key="practice.id" class="list-group-item"
              style="text-align:left">
            {{ practice.date | date }}
            {{ practice.duration | duration }}
            {{ practice.notes }}
            <button @click="deletePractice(practice.id)" class="btn btn-secondary">Delete</button>
          </li>
        </ul>
      </div>
    </template>
  </div>
</template>

<script lang="ts">
import {Component, Vue} from 'vue-property-decorator';
import moment from 'moment';
import Datepicker from 'vuejs-datepicker';

interface PracticeItem {
  id: number;
  date: Date;
  duration: number;
  notes: string;
}

@Component({
  components: {
    Datepicker
  },
  filters: {
    date(date: string): string {
      if (!date) return '';
      return moment(date).format('DD.MM.YYYY');
    },
    duration(duration: string): string {
      if (!duration) return '';
      return `${duration}min`;
    }
  }
})
export default class Practice extends Vue {
  public practices: Array<PracticeItem> = [];

  public newPracticeItem: PracticeItem = this.getEmptyPractice();

  /* used by datepicker */
  public disabledDates = {
    from: new Date()
  }

  mounted() {
    this.newPracticeItem = this.getEmptyPractice();
    if (localStorage.getItem('practiceLog')) {
      const storedItems = localStorage.getItem('practiceLog');
      this.practices = JSON.parse(storedItems || '');
    } else {
      this.practices = [];
    }
  }

  public addPractice(): void {
    this.practices.push(this.newPracticeItem);
    this.newPracticeItem = this.getEmptyPractice();
    localStorage.setItem('practiceLog', JSON.stringify(this.practices));
  }

  public deletePractice(id: number): void {
    this.practices = this.practices.filter((item: PracticeItem) => item.id !== id);
  }

  private getId(): number {
    if (this.practices && this.practices.length > 0) {
      const maxId = this.practices.reduce(
        (max: number, practice: PracticeItem) => (practice.id > max ? practice.id : max),
        ((this.practices[0]).id)
      );
      return maxId + 1;
    }
    return 1;
  }

  private getEmptyPractice(): PracticeItem {
    return {
      id: this.getId(),
      date: new Date(),
      duration: 0,
      notes: ''
    };
  }

  public customFormatter(date: string): string {
    return moment(date).format('DD.MM.YYYY');
  }
}
</script>

<style>
.practice-form {
  width: 400px;
  margin: auto;
}
.vdp-datepicker input, .vdp-datepicker input:focus {
  border: 0;
}

.form-control {
  margin-bottom: 20px;
}
</style>
