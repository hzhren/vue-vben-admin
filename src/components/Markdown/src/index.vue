<template>
  <div ref="wrapRef" />
</template>
<script lang="ts">
  import { defineComponent, ref, onMounted, unref, onUnmounted, nextTick, computed } from 'vue';
  import Vditor from 'vditor';
  import 'vditor/dist/index.css';

  import { propTypes } from '/@/utils/propTypes';
  import { useLocale } from '/@/locales/useLocale';
  import { useModalContext } from '../../Modal';

  type Lang = 'zh_CN' | 'en_US' | 'ja_JP' | 'ko_KR' | undefined;
  export default defineComponent({
    inheritAttrs: false,
    props: {
      height: propTypes.number.def(360),
      value: propTypes.string.def(''),
    },
    emits: ['change', 'get'],
    setup(props, { attrs, emit }) {
      const wrapRef = ref<ElRef>(null);
      const vditorRef = ref<Nullable<Vditor>>(null);
      const initedRef = ref(false);

      const modalFn = useModalContext();

      const lang = ref<Lang>();

      const { getLang } = useLocale();

      const getCurrentLang = computed((): 'zh_CN' | 'en_US' | 'ja_JP' | 'ko_KR' => {
        switch (unref(getLang)) {
          case 'en':
            lang.value = 'en_US';
            break;
          case 'ja':
            lang.value = 'ja_JP';
            break;
          case 'ko':
            lang.value = 'ko_KR';
            break;
          default:
            lang.value = 'zh_CN';
        }
        return lang.value;
      });
      function init() {
        const wrapEl = unref(wrapRef);
        if (!wrapEl) return;
        const bindValue = { ...attrs, ...props };
        vditorRef.value = new Vditor(wrapEl, {
          lang: unref(getCurrentLang),
          mode: 'sv',
          preview: {
            actions: [],
          },
          input: (v) => {
            // emit('update:value', v);
            emit('change', v);
          },
          blur: () => {
            unref(vditorRef)?.setValue(props.value);
          },
          ...bindValue,
          cache: {
            enable: false,
          },
        });
        initedRef.value = true;
      }

      const instance = {
        getVditor: (): Vditor => vditorRef.value!,
      };

      onMounted(() => {
        nextTick(() => {
          init();
          setTimeout(() => {
            modalFn?.redoModalHeight?.();
          }, 200);
        });

        emit('get', instance);
      });

      onUnmounted(() => {
        const vditorInstance = unref(vditorRef);
        if (!vditorInstance) return;
        try {
          vditorInstance?.destroy?.();
        } catch (error) {}
      });

      return {
        wrapRef,
        ...instance,
      };
    },
  });
</script>
