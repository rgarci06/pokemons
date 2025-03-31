<template>
  <div>
    <!-- Sección de Setup -->
    <section v-if="currentScreen === 'setup'" class="setup-container">
      <h2 class="setup-title">Configuració dels Jugadors</h2>
      <p class="setup-instructions">Introdueix els noms dels jugadors per començar el joc.</p>
      <div class="toggle-container">
        <label for="two-players-toggle">Dos Jugadors:</label>
        <label class="switch">
          <input type="checkbox" v-model="isTwoPlayers" id="two-players-toggle" />
          <span class="slider"></span>
        </label>
      </div>
      <div class="player-input-group">
        <label for="player1-name" class="player-label">Nom del Jugador 1:</label>
        <input type="text" v-model="player1Name" class="player-input" id="player1-name" required />
      </div>
      <div class="player-input-group" v-if="isTwoPlayers">
        <label for="player2-name" class="player-label">Nom del Jugador 2:</label>
        <input type="text" v-model="player2Name" class="player-input" id="player2-name" required />
      </div>
      <button @click="startGame" class="btn setup-button">Següent</button>
    </section>

    <!-- Sección de Selección de Equip -->
    <section v-if="currentScreen === 'teamSelection'" id="team-selection-section">
      <h2>Selecciona el teu Equip</h2>
      <h2>{{ currentPlayerSelectionMessage }}</h2>        
      <h2 id="credits-display">Crèdits restants: <span id="credits-value">{{ creditsDisplay }}</span></h2>
      <div id="team-section">
        <h2 id="current-player-selection">{{ currentPlayerSelectionDisplay }}</h2>
        <div id="selected-team-grid" class="grid-container" ref="teamContainer">
          <pokemon-card v-for="(poke, index) in currentPlayerTeam" :key="index" :pokemon="poke" :is-selected="isPokemonInTeam(poke.name)" @toggle-selection="handleToggleSelection" />
        </div>
      </div>
      <button id="next-player-button" @click="handleNextPlayer" class="btn">{{ buttonLabel }}</button>
      <!-- Opcions d'ordenació -->
      <div id="sort-options-section">
        <h2>Opcions d'Ordenació</h2>
        <form id="sort-options-form">
          <fieldset>
            <legend>Ordena per:</legend>
            <label><input type="radio" name="sort-criteria" value="name" v-model="sortCriteria" /> Nom</label>
            <label><input type="radio" name="sort-criteria" value="points" v-model="sortCriteria" /> Punts</label>
            <label><input type="radio" name="sort-criteria" value="type" v-model="sortCriteria" /> Tipus</label>
          </fieldset>
          <fieldset>
            <legend>Mètode d'ordenació:</legend>
            <label><input type="radio" name="sort-method" value="bubble" v-model="sortMethod" /> Bombolla</label>
            <label><input type="radio" name="sort-method" value="insertion" v-model="sortMethod" /> Inserció</label>
            <label><input type="radio" name="sort-method" value="selection" v-model="sortMethod" /> Selecció</label>
          </fieldset>
          <button type="button" id="sort-team" @click="handleSortOptions" class="btn">Ordenar</button>
        </form>
      </div>
      <div id="pokemon-grid" class="grid-container" ref="gridContainer">
        <pokemon-card v-for="(poke, index) in globalPokemonList" :key="index" :pokemon="poke" :is-selected="isPokemonInTeam(poke.name)" @toggle-selection="handleToggleSelection" />
      </div>
    </section>

    <!-- Sección de Batalla (iniciada con "Atacar!") -->
    <section id="battle-section" v-if="currentScreen === 'battleSection'">
      <h2>Moment de la Batalla!</h2>
      <p id="current-turn-display">{{ currentPlayerSelectionMessage }}</p>
      <button id="perform-attack-button" @click="startBattle" class="btn">Atacar!</button>
    </section>

    <!-- Vista General dels Equips y Arena de Batalla -->
    <section id="teams-overview-section" v-if="currentScreen === 'battleArena'">
      <h2>Vista General dels Equips</h2>
      <div id="teams-overview">
        <div class="team-container" id="player1-team-container">
          <h3>Equip del jugador {{ viewModel.player1.name }}</h3>
          <div class="team-pokemon">
            <pokemon-card v-for="(poke, index) in viewModel.player1.team.getTeam()" :key="index" :pokemon="poke" />
          </div>
        </div>
        <div class="team-container" id="player2-team-container">
          <h3>Equip del Jugador {{ viewModel.player2.name }}</h3>
          <div class="team-pokemon">
            <pokemon-card v-for="(poke, index) in viewModel.player2.team.getTeam()" :key="index" :pokemon="poke" />
          </div>
        </div>
      </div>
      <div id="battle-arena-section">
        <div class="fighter-container" id="pokemon1-display">
          <pokemon-card v-if="pokemon1Arena" :pokemon="pokemon1Arena" />
        </div>
        <div class="fighter-container" id="pokemon2-display">
          <pokemon-card v-if="pokemon2Arena" :pokemon="pokemon2Arena" />
        </div>
      </div>
      <div id="battle-vs"><p>VS</p></div>
      <div id="battle-log" class="battle-log-container">
        <h2>Registre de la Batalla</h2>
        <p v-for="(log, index) in battleLog" :key="index" :style="{ fontWeight: log.bold ? 'bold' : 'normal' }">{{ log.text }}</p>
      </div>
    </section>
  </div>
</template>

<script>
import PokemonCard from "./components/PokemonCard.vue";
import { PokemonTeamViewModel } from "./components/viewmodels/viewModel.js";

export default {
  components: {
    "pokemon-card": PokemonCard,
  },
  data() {
    return {
      currentScreen: "setup",
      isTwoPlayers: true,
      player1Name: "",
      player2Name: "",
      viewModel: new PokemonTeamViewModel(),
      currentPlayerSelectionMessage: "",
      currentPlayerSelectionDisplay: "",
      sortCriteria: "name",
      sortMethod: "bubble",
      globalPokemonList: [],
      pokemon1Arena: null,
      pokemon2Arena: null,
      battleLog: [],
      battleFinished: false,
      buttonLabel: "Següent Jugador",
    };
  },
  methods: {
    async startGame() {
      if (!this.player1Name || (this.isTwoPlayers && !this.player2Name)) {
        alert("Si us plau, introdueix els noms de tots els jugadors.");
        return;
      }
      if (!this.isTwoPlayers) {
        this.player2Name = "CPU";
      }
      this.viewModel.initializeMatch(this.player1Name, this.player2Name);
      await this.loadPokemons();
      if (this.globalPokemonList.length === 0) {
        alert("No s'han pogut carregar els Pokémon des de pokemon_data.json. Comprova que l'arxiu existeixi i tingui el format correcte.");
        this.currentScreen = "setup";
        return;
      }
      this.currentScreen = "teamSelection";
      this.startTeamSelection();
    },
    async loadPokemons() {
  try {
    const response = await fetch("../public/pokemon_data.json");
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}, statusText: ${response.statusText}`);
    }
    const text = await response.text(); // Obtener el texto de la respuesta para depurar
    console.log("Contenido de pokemon_data.json:", text); // Mostrar el contenido en la consola
    const data = JSON.parse(text); // Intentar parsear el JSON
    this.viewModel.pokemonList.loadPokemons(data);
    this.globalPokemonList = this.viewModel.getGlobalList();
  } catch (error) {
    console.error("Error carregant Pokémon des de pokemon_data.json:", error);
    console.error("Ruta intentada:", "./assets/pokemon_data.json");
    this.globalPokemonList = [];
  }
},
    startTeamSelection() {
      this.currentPlayerSelectionMessage = `${this.viewModel.player1.name}, selecciona el teu equip Pokémon`;
      this.currentPlayerSelectionDisplay = `Equip de ${this.viewModel.player1.name}`;
    },
    handleNextPlayer() {
      if (this.viewModel.currentPlayer === this.viewModel.player1) {
        if (this.viewModel.player1.team.selectedTeam.length === 0) {
          alert("Selecciona almenys un Pokémon per al teu equip!");
          return;
        }
        this.viewModel.switchPlayer();
        if (this.isTwoPlayers) {
          this.currentPlayerSelectionMessage = `${this.viewModel.player2.name}, selecciona el teu equip Pokémon`;
          this.currentPlayerSelectionDisplay = `Equip de ${this.viewModel.player2.name}`;
          this.buttonLabel = "Fi de la selecció d'equips";
        } else {
          this.viewModel.autoSelectCpuTeam();
          this.currentPlayerSelectionMessage = `${this.viewModel.player2.name} ha seleccionat el seu equip.`;
          this.buttonLabel = "Fi de la selecció d'equips";
          this.transitionToBattle();
        }
      } else if (this.viewModel.currentPlayer === this.viewModel.player2) {
        if (this.viewModel.player2.team.selectedTeam.length === 0) {
          alert("Selecciona almenys un Pokémon per al teu equip!");
          return;
        }
        this.transitionToBattle();
      }
    },
    transitionToBattle() {
      this.currentScreen = "battleSection";
      this.currentPlayerSelectionMessage = `Moment de la Batalla! Primer torn de ${this.viewModel.player1.name}`;
    },
    handleSortOptions() {
      this.viewModel.sortGlobalList(this.sortCriteria, this.sortMethod);
      this.globalPokemonList = this.viewModel.getGlobalList();
    },
    isPokemonInTeam(name) {
      return this.viewModel.currentPlayer.team.selectedTeam.some((p) => p.name === name);
    },
    handleToggleSelection(pokemon) {
      const isInTeam = this.isPokemonInTeam(pokemon.name);
      if (isInTeam) {
        this.viewModel.removePokemonFromTeam(pokemon.name);
      } else {
        if (!this.viewModel.addPokemonToCurrentPlayer(pokemon)) {
          alert(
            this.viewModel.currentPlayer.team.selectedTeam.length >= 6
              ? "L'equip ja té 6 Pokémon!"
              : "No tens suficients crèdits per afegir aquest Pokémon."
          );
        }
      }
    },
    async startBattle() {
      this.currentScreen = "battleArena";
      this.battleLog = [];
      this.battleFinished = false;
      this.battleLog.push({ text: `${this.viewModel.player1.name} vs ${this.viewModel.player2.name} ha començat!`, bold: false });

      while (
        this.viewModel.player1.team.selectedTeam.length > 0 &&
        this.viewModel.player2.team.selectedTeam.length > 0 &&
        !this.battleFinished
      ) {
        await this.fightRound();
        await new Promise(resolve => setTimeout(resolve, 5000)); // Pausa de 5 segundos entre rondas
      }

      if (!this.battleFinished) {
        this.endBattle();
      }
    },
    async fightRound() {
      // Seleccionar los Pokémon que van a luchar
      const pokemon1 = this.viewModel.getRandomFighter(this.viewModel.player1.team);
      const pokemon2 = this.viewModel.getRandomFighter(this.viewModel.player2.team);

      if (!pokemon1 || !pokemon2) {
        this.endBattle();
        return;
      }

      // Actualizar la arena con los Pokémon que están luchando
      this.pokemon1Arena = pokemon1;
      this.pokemon2Arena = pokemon2;
      this.pokemon1Arena.fights();
      this.pokemon2Arena.fights();
      this.battleLog.push({ text: `⚔️ ${pokemon1.name} vs ${pokemon2.name}`, bold: false });

      // Forzar actualización de la interfaz antes del resultado
      await this.$nextTick();

      // Resolver el combate
      if (pokemon1.special_power === pokemon2.special_power) {
        this.battleLog.push({ text: `💥 ${pokemon1.name} i ${pokemon2.name} es derroten mútuament!`, bold: true });
        this.viewModel.player1.team.removePokemon(pokemon1.name);
        this.viewModel.player2.team.removePokemon(pokemon2.name);
      } else if (pokemon1.special_power > pokemon2.special_power) {
        this.battleLog.push({ text: `💥 ${pokemon1.name} derrota ${pokemon2.name}!`, bold: false });
        const damageMade = this.viewModel.player2.team.removePokemon(pokemon2.name);
        const message = this.viewModel.player1.team.decreaseSpecialPower(pokemon1.name, damageMade);
        this.battleLog.push({ text: message, bold: true });
      } else {
        this.battleLog.push({ text: `💥 ${pokemon2.name} derrota ${pokemon1.name}!`, bold: false });
        const damageMade = this.viewModel.player1.team.removePokemon(pokemon1.name);
        const message = this.viewModel.player2.team.decreaseSpecialPower(pokemon2.name, damageMade);
        this.battleLog.push({ text: message, bold: true });
      }

      // No actualizamos la arena aquí, ya que el próximo fightRound seleccionará nuevos Pokémon
      this.$nextTick(() => {
        const log = this.$el.querySelector("#battle-log");
        if (log) log.scrollTop = log.scrollHeight;
      });
    },
    endBattle() {
      const winner = this.viewModel.player1.team.selectedTeam.length > 0
        ? this.viewModel.player1.name
        : this.viewModel.player2.name;
      this.battleLog.push({ text: `🏆 La batalla ha acabat! ${winner} és el guanyador!`, bold: true });
      this.battleFinished = true;
      this.pokemon1Arena = null;
      this.pokemon2Arena = null; // Limpiar la arena al finalizar
    },
  },
  mounted() {
    console.log("Grid container:", this.$refs.gridContainer);
  },
  computed: {
    creditsDisplay() {
      return this.viewModel.getCredits();
    },
    currentPlayerTeam() {
      return this.viewModel.getCurrentTeam();
    },
  },
};
</script>

<style>
</style>