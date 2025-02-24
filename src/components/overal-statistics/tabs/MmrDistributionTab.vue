<template>
  <div>
    <v-row align="center">
      <v-col cols="md-2">
        <v-card-text>
          <v-select
            v-model="selectedSeason"
            :items="seasons"
            item-text="id"
            @change="setSelectedSeason"
            :label="
              $t(
                `components_overall-statistics_tabs_mmrdistributiontab.selectseason`
              )
            "
            return-object
            outlined
          />

          <v-select
            v-model="selectedGameMode"
            class="over-chart-select-box"
            :items="gameModes"
            item-text="modeName"
            item-value="modeId"
            @change="gameModeChanged"
            :label="
              $t(`components_overall-statistics_tabs_mmrdistributiontab.mode`)
            "
            outlined
          />
        </v-card-text>
        <v-card-text v-if="!loadingMapAndRaceStats">
          <gateway-select @gatewayChanged="gatewayChanged" v-if="isGatewayNeeded()"/>
        </v-card-text>

        <v-card-text>
          {{
            $t(
              "components_overall-statistics_tabs_mmrdistributiontab.purplebarsdesc"
            )
          }}
        </v-card-text>
        <v-card-text v-if="authCode">
          {{
            $t(
              "components_overall-statistics_tabs_mmrdistributiontab.greenbardesc"
            )
          }}
        </v-card-text>
      </v-col>
      <v-col cols="md-10">
        <div class="text-center my-auto">
          <v-progress-circular
            indeterminate
            v-if="loadingData"
          ></v-progress-circular>
        </div>
        <mmr-distribution-chart
          v-if="!loadingData"
          :mmr-distribution="mmrDistribution"
          :selected-season="selectedSeason"
          :selected-game-mode="selectedGameMode"
        />
      </v-col>
    </v-row>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";
import { Gateways, Season } from "@/store/ranking/types";
import { SeasonGameModeGateWayForMMR } from "@/store/overallStats/types";
import { EGameMode } from "@/store/typings";
import GatewaySelect from "@/components/common/GatewaySelect.vue";
import GameModeSelect from "@/components/common/GameModeSelect.vue";
import MmrDistributionChart from "@/components/overal-statistics/MmrDistributionChart.vue";
import { Watch } from "vue-property-decorator";

@Component({
  components: { MmrDistributionChart, GameModeSelect, GatewaySelect },
})
export default class PlayerActivityTab extends Vue {
  public selectedSeason: Season = { id: 1 };
  public selectedGameMode: EGameMode = EGameMode.GM_1ON1;
  public selectedGateWay: Gateways = Gateways.Europe;
  private loadingData = true;

  get seasons() {
    return this.$store.direct.state.rankings.seasons;
  }

  get gameModes() {
    return [
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_1ON1]}`),
        modeId: EGameMode.GM_1ON1,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_2ON2]}`),
        modeId: EGameMode.GM_2ON2,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_2ON2_AT]}`),
        modeId: EGameMode.GM_2ON2_AT,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_4ON4]}`),
        modeId: EGameMode.GM_4ON4,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_FFA]}`),
        gameMode: EGameMode.GM_FFA,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_FOOTMEN_FRENZY]}`),
        modeId: EGameMode.GM_FOOTMEN_FRENZY,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_LEGION_4v4_X3]}`),
        modeId: EGameMode.GM_LEGION_4v4_X3,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_LEGION_1v1_x20]}`),
        modeId: EGameMode.GM_LEGION_1v1_x20,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_LEGION_4v4_X20]}`),
        modeId: EGameMode.GM_LEGION_4v4_X20,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_ROC_1ON1]}`),
        modeId: EGameMode.GM_ROC_1ON1,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_LTW_1ON1]}`),
        modeId: EGameMode.GM_LTW_1ON1,
      },
      {
        modeName: this.$t(`gameModes.${EGameMode[EGameMode.GM_FROSTCRAFT_4ON4]}`),
        modeId: EGameMode.GM_FROSTCRAFT_4ON4,
      }
    ]
  }
  get loadingMapAndRaceStats(): boolean {
    return this.$store.direct.state.overallStatistics.loadingMapAndRaceStats;
  }

  public async setSelectedSeason(season: Season) {
    this.loadingData = true;
    this.selectedSeason = season;
    if (this.verifiedBtag) {
      await this.$store.direct.dispatch.player.loadProfile({
        battleTag: this.verifiedBtag,
        freshLogin: false,
      });
      await this.$store.direct.dispatch.player.loadGameModeStats({
        battleTag: this.verifiedBtag,
        season: season.id,
      });
    }
    this.updateMMRDistribution();
  }
  public async updateMMRDistribution() {
    const payload: SeasonGameModeGateWayForMMR = {
      season: this.selectedSeason.id,
      gameMode: this.selectedGameMode,
      gateWay: this.selectedGateWay,
    };
    await this.$store.direct.dispatch.overallStatistics.loadMmrDistribution(
      payload
    );
    this.loadingData = false;
  }

  gameModeChanged(gameMode: EGameMode) {
    this.selectedGameMode = gameMode;
    this.updateMMRDistribution();
  }

  gatewayChanged(gateWay: Gateways) {
    this.selectedGateWay = gateWay;
    this.updateMMRDistribution();
  }

  public isGatewayNeeded() {
    return this.selectedSeason.id <= 5;
  }

  mounted() {
    this.init();
  }

  private async init() {
    await this.$store.direct.dispatch.rankings.retrieveSeasons();
    await this.setSelectedSeason(this.seasons[0]);
    this.updateMMRDistribution();
  }

  get verifiedBtag() {
    return this.$store.direct.state.oauth.blizzardVerifiedBtag;
  }

  get mmrDistribution() {
    return this.$store.direct.state.overallStatistics.mmrDistribution;
  }

  get authCode(): string {
    return this.$store.direct.state.oauth.token;
  }

  @Watch("verifiedBtag")
  async onBattleTagChanged(newBattleTag: string) {
    if (newBattleTag) {
      await this.$store.direct.dispatch.player.loadProfile({
        battleTag: newBattleTag,
        freshLogin: false,
      });
      await this.$store.direct.dispatch.player.loadGameModeStats({
        battleTag: newBattleTag,
        season: this.selectedSeason.id,
      });
    }
  }
}
</script>
