<template>
    <div v-loading="loading">
        <div>
            <el-form-item :label="$t('website.enable')">
                <el-switch v-model="data.enable" @change="updateEnable"></el-switch>
            </el-form-item>
        </div>
        <LogFile :config="{ id: id, type: 'website', name: logType }" :style="style">
            <template #button>
                <el-button @click="cleanLog" icon="Delete" :disabled="data.content === ''">
                    {{ $t('commons.button.clean') }}
                </el-button>
            </template>
        </LogFile>
    </div>
    <OpDialog ref="opRef" />
</template>
<script lang="ts" setup>
import { computed, ref } from 'vue';
import { OpWebsiteLog } from '@/api/modules/website';
import i18n from '@/lang';
import LogFile from '@/components/log-file/index.vue';
import { MsgSuccess } from '@/utils/message';

const props = defineProps({
    logType: {
        type: String,
        default: '',
    },
    id: {
        type: Number,
        default: 0,
    },
});
const logType = computed(() => {
    return props.logType;
});
const id = computed(() => {
    return props.id;
});
const style = ref('height: calc(100vh - 400px); width: 100%; min-height: 400px');
const loading = ref(false);
const data = ref({
    enable: false,
    content: '',
    path: '',
});
const opRef = ref();

const updateEnable = () => {
    const operate = data.value.enable ? 'enable' : 'disable';
    const req = {
        id: id.value,
        operate: operate,
        logType: logType.value,
    };
    loading.value = true;
    OpWebsiteLog(req)
        .then(() => {
            MsgSuccess(i18n.global.t('commons.msg.operationSuccess'));
        })
        .finally(() => {
            loading.value = false;
        });
};

const cleanLog = async () => {
    let log = logType.value === 'access.log' ? i18n.global.t('website.accessLog') : i18n.global.t('website.errLog');
    opRef.value.acceptParams({
        title: i18n.global.t('commons.msg.clean'),
        names: [],
        msg: i18n.global.t('commons.msg.operatorHelper', [log, i18n.global.t('commons.msg.clean')]),
        api: OpWebsiteLog,
        params: { id: id.value, operate: 'delete', logType: logType.value },
    });
};
</script>
