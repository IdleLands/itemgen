<template>
  <div>

    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
      <a class="navbar-brand" href="#">IdleLands Item Generator</a>
    </nav>

    <div id="app" class="container pt-3">
      <div class="row mb-3">
        <div class="col-12 text-center">
          <div class="btn-group btn-group-toggle">
            <label class="btn btn-secondary" v-bind:class="{ 'active': curType === 'item' }">
              <input type="radio" v-model="curType" id="item" value="item"> Item
            </label>
            <label class="btn btn-secondary" v-bind:class="{ 'active': curType === 'event' }">
              <input type="radio" v-model="curType" id="event" value="event"> Event
            </label>
          </div>
        </div>
      </div>

      <div class="row" v-if="curType === 'item'">
        <div class="col">
          <div class="row">
            <div class="col-3 head">Name</div>
            <div class="col-9">
              <input type="text" class="form-control" v-model.trim="form.name" />
            </div>
          </div>

          <div class="row">
            <div class="col-3 head">Slot</div>
            <div class="col-9">
              <select v-model="form.type" class="form-control">
                <option v-for="slot in slots" :value="slot">{{ slot }}</option>
              </select>
            </div>
          </div>

          <div class="row" v-for="stat in stats">
            <div class="col-3 head">{{ stat }}</div>
            <div class="col-9">
              <input type="number" class="form-control" v-model="form[stat]" />
            </div>
          </div>

          <div class="row">
            <div class="col-3 head">Item Score</div>
            <div class="col-9">{{ itemScore }}</div>
          </div>

          <div class="row">
            <div class="col-3 head">Item Tier</div>
            <div class="col-9">{{ itemTier }}</div>
          </div>

          <div class="row">
            <div class="col-3 head">Item String</div>
            <div class="col-9">{{ itemString }}</div>
          </div>
        </div>
      </div>

      <div class="row" v-if="curType === 'event'">
        <div class="col">

          <div class="row">
            <div class="col-3 head">Event Type</div>
            <div class="col-9">
              <select v-model="form.eventType" class="form-control">
                <option v-for="event in events" :value="event">{{ event }}</option>
              </select>
            </div>
          </div>

          <div class="row">
            <div class="col-3 head">Event Text</div>
            <div class="col-9">
              <input type="text" class="form-control" v-model.trim="form.eventName" />
            </div>
          </div>

          <div class="row">
            <div class="col-3 head">Event String</div>
            <div class="col-9">{{ eventString }}</div>
          </div>
        </div>
      </div>

      <div class="row">
        <div class="col-3"></div>
        <div class="col-9">
          <button class="btn btn-primary" :disabled="canStore()" @click.prevent="store()">
            Store
          </button>
          <button class="btn btn-warning float-right" @click.prevent="clear()">
            Clear
          </button>
        </div>
      </div>

      <div class="row">
        <div class="col-3 head">Stored Items</div>
        <div class="col-9">
          <div v-for="item in prevItems">{{ item }}</div>
        </div>
      </div>
    </div>
  
  </div>
</template>

<script>
const baseObj = {
  name: 'Item Name',
  type: '',
  str: 0,
  dex: 0,
  int: 0,
  con: 0,
  agi: 0,
  luk: 0,
  hp: 0,
  xp: 0,
  gold: 0,

  eventName: '%player did something.',
  eventType: ''
};

export default {
  el: '#app',
  
  data: function() {
    return {
      curType: 'item',
      events: ['battle', 'blessGold', 'blessGoldParty', 'blessItem', 'blessXp', 'blessXpParty', 'enchant', 'findItem', 'flipStat', 'forsakeGold', 'forsakeItem', 'forsakeXp', 'levelDown', 'merchant', 'party', 'providence', 'tinker', 'witch'],
      strings: [],
      slots: ['body', 'charm', 'feet', 'finger', 'hands', 'head', 'legs', 'mainhand', 'neck', 'offhand', 'suffix', 'prefix'],
      stats: ['str', 'dex', 'int', 'con', 'agi', 'luk', 'hp', 'xp', 'gold'],
      form: Object.assign({}, baseObj),

      prevItems: JSON.parse(localStorage.getItem('prevItems')) || []
    }
  },

  computed: {
    itemScore() {
      return this.form.hp  * 1
          +  this.form.str * 4
          +  this.form.int * 3
          +  this.form.dex * 1
          +  this.form.agi * 1
          +  this.form.con * 3
          +  this.form.luk * 5
          +  this.form.xp * 20
          +  this.form.gold * 10
    },

    itemTier() {
      if(this.itemScore >= 100000) return 'omega';
      if(this.itemScore >= 50000)  return 'goatly';
      if(this.itemScore >= 5000)   return 'godly';
      if(this.itemScore >= 1000)   return 'idle';
      if(this.itemScore >= 500)    return 'pro';
      if(this.itemScore >= 150)    return 'basic';
      return 'newbie';
    },

    itemString() {
      let base = `[${this.itemTier} ${this.form.type}] "${this.form.name}"`;

      this.stats.forEach(stat => {
        if(!this.form[stat]) return;

        base = `${base} ${stat}=${this.form[stat]}`;
      });

      return base;
    },

    eventString() {
      let base = `[event ${this.form.eventType}]`;
      return `${base} "${this.form.eventName}"`;
    }
  },

  methods: {
    canStore() {
      if(this.curType === 'item') 
        return !this.form.name || !this.form.type;

      if(this.curType === 'event') 
        return !this.form.eventName || !this.form.eventType;

      return false;
    },

    store() {
      let string = '';
      if(this.curType === 'item') string = this.itemString;
      if(this.curType === 'event') string = this.eventString;

      this.prevItems.push(string);
      this.form = Object.assign({}, baseObj);
      localStorage.setItem('prevItems', JSON.stringify(this.prevItems));
    },

    clear() {
      this.prevItems = [];
      localStorage.setItem('prevItems', JSON.stringify(this.prevItems));
    }
  }
}

</script>

<style>
  .row {
    margin-top: 5px;
  }

  .head {
    text-align: right;
    font-weight: bold;
  }
</style>
