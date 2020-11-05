<template>
    <div class="input">
        <div
            class="asset-wrapper"
            @click="openModal"
        >
            <div class="asset-meta">
                <AssetIcon
                    class="asset-icon"
                    :address="address"
                />
                <span class="asset-symbol">{{ symbol }}</span>
            </div>
            <Icon
                class="chevron-icon"
                :title="'chevron'"
            />
        </div>
        <div class="amount-wrapper">
            <div
                v-if="modalKey === 'input' && address !== ETH_KEY"
                class="max-button"
                @click="setMax"
            >
                MAX
            </div>
            <span v-else />
            <div
                v-if="loading"
                class="loading"
            />
            <input
                v-else
                :value="amount"
                placeholder="0.0"
                class="amount"
                @input="handleInputChange($event.target.value)"
            >
        </div>
    </div>
</template>

<script lang="ts">
import BigNumber from 'bignumber.js';
import { defineComponent, computed } from 'vue';
import { useStore } from 'vuex';

import { scale } from '@/utils/helpers';

import AssetIcon from '@/components/AssetIcon.vue';
import Icon from '@/components/Icon.vue';

export default defineComponent({
    components: {
        AssetIcon,
        Icon,
    },
    props: {
        modalKey: {
            type: String,
            required: true,
        },
        address: {
            type: String,
            required: true,
        },
        amount: {
            type: String,
            required: true,
        },
        loading: {
            type: Boolean,
            default: false,
        },
    },
    emits: ['update:amount', 'change'],
    setup(props, { emit }) {
        const store = useStore();

        const symbol = computed(() => {
            const assets = store.state.assets.metadata;
            const asset = assets[props.address];
            if (!asset) {
                return '';
            }
            return asset.symbol;
        });

        function setMax(): void {
            const assets = store.state.assets.metadata;
            const { balances } = store.state.account;
            const balance = balances[props.address];
            const assetDecimals = assets[props.address].decimals;
            const balanceNumber = new BigNumber(balance);
            const amountNumber = scale(balanceNumber, -assetDecimals);
            const amount = amountNumber.toString();
            handleInputChange(amount);
        }

        function handleInputChange(value: string): void {
            emit('change', value);
            emit('update:amount', value);
        }

        function openModal(): void {
            store.dispatch('ui/openAssetModal', props.modalKey);
        }

        return {
            symbol,
            setMax,
            handleInputChange,
            openModal,
        };
    },
});
</script>

<style scoped>
.input {
    display: flex;
    height: 80px;
    border: 1px solid var(--outline);
    border-radius: 4px;
    background: var(--background-secondary);
}

.amount-wrapper {
    width: 240px;
    padding: 8px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    border-right: 1px solid var(--outline);
    border-radius: 4px;
}

.loading {
    width: 80px;
    height: 21px;
    background: var(--text-primary);
    animation-name: pulse;
    animation-duration: 2s;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
}

@keyframes pulse {
    0% {
        opacity: 0.2;
    }

    10% {
        opacity: 0.7;
    }

    100% {
        opacity: 0.2;
    }
}

.max-button {
    color: var(--info);
    font-size: 12px;
    border: 1px solid;
    padding: 2px;
    cursor: pointer;
    border-radius: 4px;
}

.amount {
    border: none;
    background: transparent;
    color: var(--text-primary);
    font-size: 24px;
    width: 200px;
    text-align: right;
    outline: none;
}

.asset-wrapper {
    width: 100px;
    display: flex;
    padding-left: 20px;
    align-items: center;
    justify-content: space-between;
    cursor: pointer;
}

.asset-wrapper:hover {
    background: var(--background-primary);
    border-radius: 0 4px 4px 0;
}

.asset-meta {
    display: flex;
    align-items: center;
}

.asset-icon {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    margin-left: 8px;
}

.asset-symbol {
    margin-left: 8px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: 88px;
}

.chevron-icon {
    width: 16px;
    height: 16px;
    margin-right: 8px;
}

@media only screen and (max-width: 768px) {
    .amount-wrapper {
        width: 160px;
    }

    .amount {
        width: 120px;
    }
}
</style>
