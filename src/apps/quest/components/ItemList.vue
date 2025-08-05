<template>
    <section class="quarter-line__selector">
        <div @click="currentSlide(1)" :class="{'active-slide': slideIndex === 1}" class="quarter-line__selector-tab">
            <img src="/quest-images/Quarters/AllItems.png" class="modal__header-image" />
            <span> All Items </span>
            <img src="/quest-images/Quarters/AllItems.png" class="modal__header-image" />
        </div>

        <div @click="currentSlide(2)" :class="{'active-slide': slideIndex === 2}" class="quarter-line__selector-tab">
            <img src="/quest-images/Quarters/AllMissions.png" class="modal__header-image" />
            <span> Mission Items </span>
            <img src="/quest-images/Quarters/AllMissions.png" class="modal__header-image" />
        </div>

        <div @click="currentSlide(3)" :class="{'active-slide': slideIndex === 3}" class="quarter-line__selector-tab">
            <img src="/quest-images/Quarters/QuarterLevel.png" class="modal__header-image" />
            <span> Quarter Items </span>
            <img src="/quest-images/Quarters/QuarterLevel.png" class="modal__header-image" />
        </div>
    </section>

    <!-- All Items Section -->
    <div class="item-list__container-inner fade">
        <p v-if="Object.keys(currentItems).length > 0">Complete missions and upgrades to update the list</p>
        <p v-else class="complete">You have completed every mission and upgrade.</p>

        <div class="search-container" v-if="Object.keys(currentItems).length > 0">
            <input type="text" v-model="searchValue" placeholder="Search for items..." />
        </div>

        <section class="list__container">
            <div
                v-for="(amount, index) in currentItems"
                :key="index.toString()"
                class="item__row"
                :class="{
                  matching: rowColor(index.toString()),
                  completed: isCompletedFor(index.toString(), currentItems)
                }"
            >
                <img :src="'/map-images/item-images/' + itemName(index.toString(), true) + '.png'" class="item__image" />

                <!-- Tracker Input placed in the middle column -->
                <input
                    type="number"
                    v-model.number="itemTracker[index]"
                    min="0"
                    class="item-tracker-input"
                    placeholder="0"
                    v-if="amount > 0" />

                <span>
                    <span v-if="index.toString() === 'SoftCurrency'"> {{ amount / 1000 }}k </span>
                    <span v-else> {{ amount }}</span>
                    {{ itemName(index.toString()) }}
                </span>
            </div>
        </section>
    </div>

    <!-- Mission Items Section -->
    <div class="item-list__container-inner fade">
        <p v-if="Object.keys(currentMissionsItems).length > 0">Complete missions to update the list</p>
        <p v-else class="complete">You have completed every mission.</p>

        <div class="search-container" v-if="Object.keys(currentMissionsItems).length > 0">
            <input type="text" v-model="searchValue" placeholder="Search for items..." />
        </div>

        <section class="list__container">
            <div
                v-for="(amount, index) in currentMissionsItems"
                :key="index.toString()"
                class="item__row"
                :class="{
                  matching: rowColor(index.toString()),
                  completed: isCompletedFor(index.toString(), currentMissionsItems)
                }"
            >
                <img :src="'/map-images/item-images/' + itemName(index.toString(), true) + '.png'" class="item__image" />

                <!-- Tracker Input for mission items -->
                <input
                    type="number"
                    v-model.number="itemTracker[index]"
                    min="0"
                    class="item-tracker-input"
                    placeholder="0"
                    v-if="amount > 0" />

                <span>
                    <span v-if="index.toString() === 'SoftCurrency'"> {{ amount / 1000 }}k </span>
                    <span v-else> {{ amount }}</span>
                    {{ itemName(index.toString()) }}
                </span>
            </div>
        </section>
    </div>

    <!-- Quarter Items Section -->
    <div class="item-list__container-inner fade">
        <p v-if="Object.keys(currentQuarterItems).length > 0">Complete upgrades to update the list</p>
        <p v-else class="complete">You have completed every upgrade.</p>

        <div class="search-container" v-if="Object.keys(currentQuarterItems).length > 0">
            <input type="text" v-model="searchValue" placeholder="Search for items..." />
        </div>

        <section class="list__container">
            <div
                v-for="(amount, index) in currentQuarterItems"
                :key="index.toString()"
                class="item__row"
                :class="{
                  matching: rowColor(index.toString()),
                  completed: isCompletedFor(index.toString(), currentQuarterItems)
                }"
            >
                <img :src="'/map-images/item-images/' + itemName(index.toString(), true) + '.png'" class="item__image" />

                <!-- Tracker Input for quarter items -->
                <input
                    type="number"
                    v-model.number="itemTracker[index]"
                    min="0"
                    class="item-tracker-input"
                    placeholder="0"
                    v-if="amount > 0" />

                <span>
                    <span v-if="index.toString() === 'SoftCurrency'"> {{ amount / 1000 }}k </span>
                    <span v-else> {{ amount }}</span>
                    {{ itemName(index.toString()) }}
                </span>
            </div>
        </section>
    </div>
</template>

<script lang="ts">
import { keyCardInfo } from "../../map/mapConstants";
import { defineComponent } from "vue";
import { POSITION, useToast } from "vue-toastification";
import { missionData, stringTables, techLevelsData, techTreeData } from "../data";
import { missions } from "../QuestConstants";
import { factionProgress, quarterProgress } from "../trackProgress";
import { getItemsOfMission, itemName } from "../utils";

export default defineComponent({
    data() {
        return {
            progressData: factionProgress,
            quarterProgress: quarterProgress,
            currentMissionsItems: {} as any,
            previousMissionItems: {} as any,
            currentQuarterItems: {} as any,
            previousQuarterItems: {} as any,
            currentItems: {} as any,
            previousItems: {} as any,
            keycardInfo: keyCardInfo,
            stringTables: stringTables["Materials"],
            toast: useToast(),
            techLevelsData: techLevelsData,
            techTreeData: techTreeData,
            slideIndex: 1,
            searchValue: "",
            itemTracker: {} // stores counts keyed by item id/name
        };
    },
    mounted() {
        this.getIncompleteParts();
        this.getQuarterItems();
        this.updateList();
        this.showSlides(this.slideIndex);
    },
    methods: {
        rowColor(name: string): boolean {
            if (this.searchValue.length < 3) return false;
            const current = itemName(name);
            return current.toLowerCase().includes(this.searchValue.toLowerCase());
        },
        plusSlides(n: number) {
            this.showSlides((this.slideIndex += n));
        },
        currentSlide(n: number) {
            this.showSlides((this.slideIndex = n));
        },
        showSlides(n: number) {
            let slides = document.getElementsByClassName("item-list__container-inner") as any;
            if (n > slides.length) this.slideIndex = 1;
            if (n < 1) this.slideIndex = slides.length;
            for (let i = 0; i < slides.length; i++) {
                slides[i].style.display = "none";
            }
            slides[this.slideIndex - 1].style.display = "block";
        },
        getIncompleteParts(): void {
            let newData: any = {};
            for (let f in missions) {
                for (let m in missions[f]) {
                    const length = missions[f][m].length;
                    const progress = this.progressData.get()[f][m];
                    if (progress >= length) {
                        // mission completed
                    } else {
                        for (let p in missions[f][m]) {
                            if (progress < parseInt(p) + 1) {
                                const partData = missionData[missions[f][m][p]];
                                if (partData) {
                                    let data = getItemsOfMission(partData["objectives"]);
                                    for (let item in data) {
                                        newData[item] = (newData[item] || 0) + data[item];
                                    }
                                }
                            }
                        }
                    }
                }
            }
            const sortedData = Object.keys(newData)
                .sort((a, b) => newData[b] - newData[a])
                .reduce((r, k) => ({ ...r, [k]: newData[k] }), {});
            this.previousMissionItems = { ...this.currentMissionsItems };
            this.currentMissionsItems = sortedData;
        },
        getQuarterItems(): void {
            let newData: any = {};
            let index = 0;
            for (let level in this.techLevelsData) {
                index += 1;
                if (this.quarterProgress.get()["overall"] < index) {
                    for (let i in this.techLevelsData[level]["costs"]) {
                        newData[i] = (newData[i] || 0) + this.techLevelsData[level]["costs"][i];
                    }
                }
            }
            for (let upgrade in this.techTreeData) {
                const levels = this.techTreeData[upgrade]["levels"];
                const progress = this.quarterProgress.get()[upgrade];
                if (upgrade >= levels.length) continue;
                for (let l in levels) {
                    if (progress < parseInt(l) + 1) {
                        for (let i in levels[l]["costs"]) {
                            newData[i] = (newData[i] || 0) + levels[l]["costs"][i];
                        }
                    }
                }
            }
            const sortedData = Object.keys(newData)
                .sort((a, b) => newData[b] - newData[a])
                .reduce((r, k) => ({ ...r, [k]: newData[k] }), {});
            this.previousQuarterItems = { ...this.previousQuarterItems };
            this.currentQuarterItems = sortedData;
        },
        sendNotification(): void {
            const before = JSON.stringify(this.previousItems);
            const after = JSON.stringify(this.currentItems);
            if (before === after) return;
            this.toast.info("Item list updated", { timeout: 2000, position: POSITION.BOTTOM_RIGHT });
        },
        updateList() {
            let bigList: any = {};
            for (let item in this.currentMissionsItems) {
                bigList[item] = (bigList[item] || 0) + this.currentMissionsItems[item];
            }
            for (let item in this.currentQuarterItems) {
                bigList[item] = (bigList[item] || 0) + this.currentQuarterItems[item];
            }
            const sortedData = Object.keys(bigList)
                .sort((a, b) => bigList[b] - bigList[a])
                .reduce((r, k) => ({ ...r, [k]: bigList[k] }), {});
            this.previousItems = { ...this.currentItems };
            this.currentItems = sortedData;
            this.sendNotification();
        },
        itemName(item: string, urlFormat?: boolean): string {
            return itemName(item, urlFormat);
        },
        // New method: checks if the user-entered amount meets/exceeds the required amount for a given list.
        isCompletedFor(key: string, list: any): boolean {
            const required = list[key] || 0;
            const entered = this.itemTracker[key] || 0;
            return entered >= required;
        }
    },
    watch: {
        progressData: {
            deep: true,
            handler() {
                this.getIncompleteParts();
                this.updateList();
            },
        },
        quarterProgress: {
            deep: true,
            handler() {
                this.getQuarterItems();
                this.updateList();
            },
        },
    },
});
</script>

<style scoped>
p {
    text-align: center;
    font-size: var(--space-xl);
    margin-bottom: 0.3rem;
}

p.complete {
    border-bottom: none;
    margin-bottom: none;
}
.list__container {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr 1fr;
    gap: var(--space-md);
}

.item__row {
    display: grid;
    grid-template-columns: auto min-content 1fr;
    gap: 0.5rem;
    align-items: center;
    height: var(--space-xl);
    position: relative;
    /* add transition for smooth overlay effect */
    transition: background-color 0.3s ease;
}

.item__row.completed {
    background-color: rgba(0, 128, 0, 0.2);
}

.item__row span {
    text-overflow: ellipsis;
    display: flex;
    align-items: center;
    gap: var(--space-xs);
}

.item__image {
    height: var(--space-xl);
}

@media screen and (max-width: 1500px) {
    .list__container {
        grid-template-columns: 1fr 1fr 1fr;
    }
}
@media screen and (max-width: 1200px) {
    .list__container {
        grid-template-columns: 1fr 1fr;
    }
}
@media screen and (max-width: 900px) {
    .list__container {
        grid-template-columns: 1fr;
        margin: auto;
    }
}

.comment {
    text-align: center;
}

.fade {
    animation-name: fade-in;
    animation-duration: 1s;
    animation-timing-function: cubic-bezier(0.23, 1, 0.32, 1);
}

@keyframes fade-in {
    from {
        opacity: 0;
        translate: 0 30%;
    }
    to {
        opacity: 1;
        translate: 0 0;
    }
}

.quarter-line__selector {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    width: 100%;
    text-align: center;
    margin: 0 0 var(--space-md);
    cursor: pointer;
    border-collapse: collapse;
}

.quarter-line__selector div {
    text-transform: uppercase;
    letter-spacing: 0.4rem;
    padding: 0.5rem;
    position: relative;
    border-collapse: collapse;
    border: 1px solid var(--border-color-base);
}

@media screen and (max-width: 900px) {
    .quarter-line__selector {
        grid-template-columns: 1fr 1fr;
    }
}

.quarter-line__selector-tab {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    text-align: center;
    gap: 1rem;
}

.quarter-line__selector-tab span {
    text-align: center;
}

.quarter-line__selector-tab img {
    width: 2rem;
}

@media screen and (max-width: 900px) {
    .quarter-line__selector-tab img {
        display: none;
    }
}

.quarter-line__selector div:not(.active-slide)::before {
    content: "";
    position: absolute;
    top: 3px;
    left: 0;
    width: 100%;
    height: 100%;
    border-bottom: 2px solid transparent;
    transition: 0.4s ease-in-out;
}

.quarter-line__selector div:hover:not(.active-slide)::before {
    border-bottom-color: var(--color-base--subtle);
}

.active-slide::before:hover {
    border-bottom-color: var(--color-base--subtle) !important;
}

.active-slide::before {
    content: "";
    position: absolute;
    top: 3px;
    left: 0;
    width: 100%;
    height: 100%;
    border-bottom: 2px solid var(--color-base--subtle);
    transition: 0.4s ease-in-out;
}

.active-slide:hover::before {
    border-bottom-color: var(--color-base--subtle);
}

.matching {
    transition: all 0.2s linear;
    background-color: var(--color-surface-1);
    color: var(--color-base--subtle);
}

.search-container {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    padding-bottom: 1rem;
    margin-bottom: 1rem;
    border-bottom: 1px solid var(--border-color-base);
}

.search-container input {
    width: 30%;
    height: 100%;
    font-size: 1.2rem;
    padding: 0.2rem;
}

.item-tracker-input {
    width: 40px;
    height: 1.8rem;
    font-size: 0.9rem;
    padding: 0.2rem;
    border: 1px solid var(--border-color-base);
    border-radius: 4px;
    text-align: center;
    margin-left: 0.5rem;
}
</style>