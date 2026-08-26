<script>
import { mapGetters } from 'vuex';
import { useAlert } from 'dashboard/composables';
import { useConversationLabels } from 'dashboard/composables/useConversationLabels';
import NextButton from 'dashboard/components-next/button/Button.vue';

// Contrato com a automacao de IA externa: enquanto esta etiqueta estiver
// aplicada na conversa, a IA nao deve responder. O titulo precisa existir em
// Configuracoes > Etiquetas, senao o botao nao e renderizado.
const AI_PAUSE_LABEL = 'ia_pausada';

export default {
  components: {
    NextButton,
  },
  setup() {
    const { savedLabels, accountLabels, onUpdateLabels } =
      useConversationLabels();

    return { savedLabels, accountLabels, onUpdateLabels };
  },
  computed: {
    ...mapGetters({
      uiFlags: 'conversationLabels/getUIFlags',
      currentChat: 'getSelectedChat',
    }),
    isPaused() {
      return this.savedLabels.includes(AI_PAUSE_LABEL);
    },
    labelExists() {
      return this.accountLabels.some(({ title }) => title === AI_PAUSE_LABEL);
    },
    // O getter getConversationLabels devolve [] tanto para "conversa sem
    // etiquetas" quanto para "etiquetas ainda nao carregadas" (o catch de
    // conversationLabels/get nao escreve o registro). Como update_labels
    // substitui a lista inteira no backend, agir sobre uma lista nao
    // carregada apagaria todas as etiquetas da conversa. Por isso lemos o
    // registro cru, onde os dois casos sao distinguiveis.
    labelsLoaded() {
      const { records } = this.$store.state.conversationLabels;
      return Array.isArray(records[Number(this.currentChat?.id)]);
    },
    isDisabled() {
      return this.uiFlags.isUpdating || !this.labelsLoaded;
    },
    buttonIcon() {
      return this.isPaused ? 'i-ph-play' : 'i-ph-user-switch';
    },
    buttonLabel() {
      return this.isPaused ? 'Devolver para IA' : 'Assumir conversa';
    },
  },
  methods: {
    async onClick() {
      const shouldPause = !this.isPaused;

      // A lista e montada a partir de savedLabels (label_list cru do servidor)
      // e nao de activeLabels, que descartaria etiquetas nao cadastradas.
      const labels = shouldPause
        ? [...new Set([...this.savedLabels, AI_PAUSE_LABEL])]
        : this.savedLabels.filter(label => label !== AI_PAUSE_LABEL);

      await this.onUpdateLabels(labels);

      // conversationLabels/update engole a excecao e apenas seta isError num
      // uiFlag global do modulo, compartilhado com o sidebar de etiquetas.
      // Conferir o estado real (reescrito com o payload da resposta) e
      // imune a uma operacao concorrente do sidebar.
      if (this.isPaused !== shouldPause) {
        useAlert('Não foi possível atualizar a conversa. Tente novamente.');
      }
    },
  },
};
</script>

<!-- eslint-disable-next-line vue/no-root-v-if -->
<template>
  <NextButton
    v-if="labelExists"
    type="button"
    :icon="buttonIcon"
    :label="buttonLabel"
    :variant="isPaused ? 'solid' : 'faded'"
    :color="isPaused ? 'amber' : 'slate'"
    :is-loading="uiFlags.isUpdating"
    :disabled="isDisabled"
    :aria-pressed="isPaused"
    sm
    @click="onClick"
  />
</template>
