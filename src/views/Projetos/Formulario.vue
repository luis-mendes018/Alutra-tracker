<template>
  <section>
    <form @submit.prevent="salvar">
      <div class="field">
        <label for="nomeDoProjeto" class="label"> Nome do projeto </label>
        <input
          type="text"
          class="input"
          v-model="nomeDoProjeto"
          id="nomeDoProjeto"
        />
      </div>
      <div class="field">
        <button class="button" type="submit">Salvar</button>
      </div>
    </form>
  </section>
</template>

<script lang="ts">
import { TipoNotificacao } from "@/interfaces/INotificacao";
import { useStore } from "@/store";
import { defineComponent, ref } from "vue";
import useNotificador from "@/hooks/notificador";
import { ALTERAR_PROJETO, CADASTRAR_PROJETO } from "@/store/tipo-acoes";
import { useRouter } from "vue-router";

export default defineComponent({
  name: "Formulario",
  props: {
    id: {
      type: String,
    },
  },
  // mounted() {
  //   if (this.id) {
  //     // Verificar se o projeto já está na store
  //     const projeto = this.store.state.projeto.projetos.find((proj) => proj.id == this.id);
  //     if (projeto) {
  //       this.nomeDoProjeto = projeto.nome;
  //     } else {
  //       // Caso o projeto não seja encontrado na store, buscar do servidor
  //       this.buscarProjeto();
  //     }
  //   }
  // },
  // data() {
  //   return {
  //     nomeDoProjeto: "",
  //   };
  // },

  setup(props) {

    const router = useRouter()

    const store = useStore();
    const { notificar } = useNotificador();

    const nomeDoProjeto = ref("");

    if (props.id) {
      const projeto = store.state.projeto.projetos.find(
        (proj) => proj.id == props.id
      );
      nomeDoProjeto.value = projeto?.nome || "";
    }

    // Função para buscar o projeto do servidor, caso não esteja na store
   const buscarProjeto = () => {
      store.dispatch("OBTER_PROJETOS").then(() => {
        const projeto = store.state.projeto.projetos.find(
          (proj) => proj.id == props.id
        );
        if (projeto) {
          nomeDoProjeto.value = projeto.nome;
        }
      });
    }

   const lidarComSucesso = () => {
      nomeDoProjeto.value = ""; // Limpar o campo
      notificar(
        TipoNotificacao.SUCESSO,
        "Excelente!",
        "Projeto salvo com sucesso"
      );
      router.push("/projetos");
    }

    const salvar = () => {
      if (props.id) {
        // Alterar o projeto existente
        store
          .dispatch(ALTERAR_PROJETO, {
            id: props.id,
            nome: nomeDoProjeto.value,
          })
          .then(() => {
            lidarComSucesso();
          })
          .catch((erro) => {
            console.error("Erro ao alterar projeto:", erro);
          });
      } else {
        // Cadastrar um novo projeto
        store.dispatch(CADASTRAR_PROJETO, nomeDoProjeto.value)
        .then(() => {lidarComSucesso();});
      }
    };

    return {
      nomeDoProjeto,
      salvar,
      buscarProjeto
    };
  },
});
</script>
