<template>
  <body>
      <header>
        <h1>Monster Slayer</h1>
      </header>
      <section id="monster" class="container">
        <h2>Monster Health</h2>
        <div class="healthbar">
          <div class="healthbar__value" :style="calcMonsterHealth"></div>
        </div>
      </section>
      <section id="player" class="container">
        <h2>Your Health</h2>
        <div class="healthbar">
          <div class="healthbar__value" :style="calcPlayerHealth"></div>
        </div>
      </section>
      <section class="container" v-if="winner">
        <h2>Game Over!</h2>
        <h3 v-if="winner === 'draw'">It's a draw 😲</h3>
        <h3 v-else-if="winner === 'player'">Hurray, you won 🥳</h3>
        <h3 v-else>Oops, you lost 😕</h3>
        <h2>Want to play again ?</h2>
        <button @click="reset">Click me to play again!</button>
      </section>
      <section id="controls">
        <button :disabled="winner" @click="attack">ATTACK</button>
        <button :disabled="winner || disableSplAtk" @click="specialAtK()">SPECIAL ATTACK</button>
        <button :disabled="winner" @click="heal">HEAL</button>
        <button :disabled="winner" @click="surrender">SURRENDER</button>
      </section>
      <section id="log" class="container">
        <h2>Battle Log</h2>
        <ul v-for="log in battleLogs" :key="log">
          <li>
            <span :class="{'log--player': log.actionBy === 'Player', 'log--monster': log.actionBy === 'Monster'}">
              {{log.actionBy}}
            </span>
            <span v-if="log.actionType === 'Heal'">
              heals himself for <span class="log--heal">{{log.actionValue}}</span>
            </span>
            <span v-else>
              attacks and deals <span class="log--damage">{{log.actionValue}}</span>
            </span>
          </li>
        </ul>
      </section>
  </body>
</template>


<script>
  const getValue = (max, min) => Math.floor(Math.random() * (max - min) + min);  

  export default {
    data() {
        return {
            playerHealth: 100,
            monsterHealth: 100,
            round: 0,
            winner: null,
            battleLogs: []
        }
    },
    watch: {
        playerHealth(value) {
            if(value <= 0 && this.monsterHealth <= 0) this.winner = 'draw';
            else if (value <= 0) this.winner = 'monster';
        },
        monsterHealth(value) {
            if(value <= 0 && this.playerHealth <= 0) this.winner = 'draw';
            else if (value <= 0) this.winner = 'player';
        }
    },
    computed: {
        calcPlayerHealth() {
            if(this.playerHealth < 0) return {width: '0%'};
            else return {width: this.playerHealth + '%'};
        },
        calcMonsterHealth() {
            if (this.monsterHealth < 0) return {width: '0%'};
            else return {width: this.monsterHealth + '%'};
        },
        disableSplAtk() {
            if(this.round === 0 || this.round % 3 !== 0) return true
            else return false;
        }
    },
    methods: {
        attack() {
            const damageCaused = getValue(12, 5);
            this.monsterHealth -= damageCaused;
            this.round ++;
            this.monsterAtk();
            this.battleLog('Player', 'Attack', damageCaused);
        },
        monsterAtk() {
            const damageCaused = getValue(18, 5);
            this.playerHealth -= damageCaused;
            this.battleLog('Monster', 'Attack', damageCaused);
        },
        specialAtK() {
            const damageCaused = getValue(25, 10);
            this.monsterHealth -= damageCaused;
            this.round ++;
            this.monsterAtk();
            this.battleLog('Player', 'Special Attack', damageCaused);
        },
        heal() {
            const healHP = getValue(8, 20);

            if(healHP + this.playerHealth > 100) this.playerHealth = 100;
            else this.playerHealth += healHP;

            this.round ++;
            this.monsterAtk();
            this.battleLog('Player', 'Heal', healHP);
        },
        reset() {
            this.battleLogs = [];
            this.round = 0;
            this.winner = null;
            this.playerHealth = 100;
            this.monsterHealth = 100
        },
        surrender() {
            this.winner = 'monster';
            this.playerHealth = 0;
            this.battleLog('Player', 'Surrender');
        },
        battleLog(who, what, value) {
            this.battleLogs.unshift({
                actionBy: who,
                actionType: what,
                actionValue: value
            });
        }
    }
  }
</script>

<style> 

  body {
    box-sizing: border-box;
    font-family: 'Jost', sans-serif;
    margin: 0;
  }

  header {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
    padding: 0.5rem;
    background-color: #880017;
    color: white;
    text-align: center;
    margin-bottom: 2rem;
    margin-top: 0;
  }

  section {
    width: 90%;
    max-width: 40rem;
    margin: auto;
  }

  .healthbar {
    width: 100%;
    height: 40px;
    border: 1px solid #575757;
    margin: 1rem 0;
    background: #fde5e5;
  }

  .healthbar__value {
    background-color: #00a876;
    width: 100%;
    height: 100%;
  }

  .container {
    text-align: center;
    padding: 0.5rem;
    margin: 1rem auto;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
    border-radius: 12px;
  }

  #monster h2,
  #player h2 {
    margin: 0.25rem;
  }

  #controls {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    align-items: center;
    justify-content: center;
  }

  button {
    font: inherit;
    border: 1px solid #88005b;
    background-color: #88005b;
    color: white;
    padding: 1rem 2rem;
    border-radius: 12px;
    margin: 1rem;
    width: 12rem;
    cursor: pointer;
    box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.26);
  }

  button:focus {
    outline: none;
  }

  button:hover,
  button:active {
    background-color: #af0a78;
    border-color: #af0a78;
    box-shadow: 1px 1px 8px rgba(0, 0, 0, 0.26);
  }

  button:disabled {
    background-color: #ccc;
    border-color: #ccc;
    box-shadow: none;
    color: #3f3f3f;
    cursor: not-allowed;
  }

  #log ul {
    list-style: none;
    margin: 0;
    padding: 0;
  }

  #log li {
    margin: 0.5rem 0;
  }

  .log--player {
    color: #7700ff;
  }

  .log--monster {
    color: #da8d00;
  }

  .log--damage {
    color: red;
  }

  .log--heal {
    color: green;
  }
</style>