<template>
  <BaseForm @submit.native.prevent="handleSubmit" class="form">
    <header>
      <h2 class="title">
        Create a Deployment
      </h2>
    </header>
    <div class="control-group">
      <div class="control-label">
        <label>Type</label>
      </div>
      <div class="controls">
        <BaseRadioButtons
          v-model="action"
          name="action"
          :options="{ promote: 'Promote', rollback: 'Rollback'}"
        />
      </div>
    </div>
    <div class="control-group">
      <div class="control-label">
        <label>Target</label>
      </div>
      <div class="controls">
        <Autocomplete v-model="target" :options="targets" />
      </div>
    </div>
    <div class="control-group">
      <label class="control-label">Parameters</label>
      <div class="controls param-list-container">
        <ul class="param-list">
          <li v-for="(val, key) in params" :key="key" class="param-list-item">
            <code>{{key}}={{val}}</code>
            <Button @click.native.prevent="handleRmParam(key)" type="button" theme="danger" size="m" outline borderless>Remove</Button>
          </li>
        </ul>
        <BaseForm @submit.native.prevent="handleAddParam" class="param-list-form">
          <BaseInput
            v-model="paramInput.key"
            name="key"
            autocomplete="off"
            autocorrect="off"
            autocapitalize="off"
            spellcheck="false"
            placeholder="key"
            type="text"
          />
          <BaseInput
            v-model="paramInput.value"
            name="value"
            autocomplete="off"
            autocorrect="off"
            autocapitalize="off"
            spellcheck="false"
            placeholder="value"
            type="text"
          />
          <Button type="submit" size="l" theme="primary" outline borderless>Add</Button>
        </BaseForm>
      </div>
    </div>
    <div class="control-actions">
      <Button type="submit" size="l" theme="primary" :loading="submitting">Deploy</Button>
      <Button type="button" size="l" outline @click.native="handleCancel">Cancel</Button>
      <div class="error-message" v-if="errors.length">{{ errors.join("\n") }}</div>
    </div>
  </BaseForm>
</template>

<script>
import Button from "@/components/buttons/Button.vue";
import BaseRadioButtons from "@/components/forms/BaseRadioButtons.vue";
import BaseInput from "@/components/forms/BaseInput.vue";
import BaseForm from "@/components/forms/BaseForm.vue";
import Autocomplete from "@/components/autocomplete/Autocomplete.vue";

export default {
  name: "DeploymentForm",
  components: {
    Button,
    BaseRadioButtons,
    BaseInput,
    BaseForm,
    Autocomplete
  },
  data() {
    return {
      errors: [],
      action: "promote",
      target: "",
      params: {},
      submitting: false,
      paramInput: {
        key: "",
        value: ""
      },
    }
  },
  computed: {
    slug() {
      return this.$route.params.namespace + "/" + this.$route.params.name;
    },
    targets() {
      return this.$store.state.deployments[this.slug]
        && this.$store.state.deployments[this.slug].data
        && Object.keys(this.$store.state.deployments[this.slug].data)
        || [];
    },
  },
  methods: {
    async handleSubmit(e) {
      this.errors = [];

      const { namespace, name, build } = this.$route.params;

      const inputs = {
        action: this.action,
        target: this.target,
        params: this.params
      };

      const deployment = {
        namespace,
        name,
        build,
        ...inputs
      };

      if (!this.target.length) this.errors.push("Target is required")
      if (!this.errors.length) {
        this.submitting = true;

        try {
          const data = await this.$store.dispatch("createDeployment", deployment)

          if (!data) {
            this.errors.push("Target not found")
          } else {
            this.$emit("submit", inputs);
            this.$router.push(`/${namespace}/${name}/${data.build.number}`);
          }
        } catch (err) {
          this.errors.push(err);
        } finally {
          this.submitting = false;
        }
      }
    },
    handleCancel(e) {
      this.$emit("cancel");
    },
    handleAddParam(e) {
      if (this.paramInput.key == "" || this.paramInput.value == "") return;
      this.params[this.paramInput.key] = this.paramInput.value;
      this.paramInput.key = "";
      this.paramInput.value = "";
    },
    handleRmParam(key) {
      this.$delete(this.params, key);
    },
  },
  mounted() {
    this.$store.dispatch("fetchDeployments", this.$store.state.route.params);
  }
};
</script>

<style scoped lang="scss">
.text {
  padding: 1rem;
}

.form {
  max-width: 520px;

  .base-input {
    width: 100%;
  }

  header {
    height: 50px;
    line-height: 50px;
    border-bottom: 1px solid rgba(30,55,90,.05);
    padding: 0 15px;
    display: flex;
    align-items: center;
    .title {
      font-size: 16px;
      font-weight: 600;
    }
  }
  .control-group,
  .control-actions,
  .header {
    padding: 11px 15px;
  }

  .control-group {
    align-items: baseline;

    .control-label {
      flex-basis: 0;
      flex-grow: 1;
      flex-shrink: 0;
      text-align: right;
      margin-right: 1.5rem;
    }
    .controls {
      flex-basis: 0;
      flex-grow: 5;
      flex-shrink: 1;
    }
  }
  .control-actions {
    .button + .button {
      margin-left: 1rem;
    }
  }
  .param-list {
    &:not(:empty) {
      margin-bottom: 16px;
    }
  }
  .param-list-item {
    display: flex;
    align-items: baseline;
    justify-content: space-between;
    padding: 2px 0;
    code {
      font-family: monospace;
    }
  }

  .param-list-form {
    display: flex;
    flex-wrap: nowrap;

    * + * {
      margin-left: 0.5rem;
    }

    * {
      flex-shrink: 1;
    }
  }
}
</style>