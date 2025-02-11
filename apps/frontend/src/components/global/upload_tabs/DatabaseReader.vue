<template>
  <v-container class="mx-0 px-0" fluid style="height: 80vh">
    <v-card-subtitle>
      View files loaded into your organizations Heimdall Server instance.
    </v-card-subtitle>
    <LoadFileList
      :headers="headers"
      :files="files"
      :loading="loading"
      @load-selected="load_results($event)"
    />
  </v-container>
</template>

<script lang="ts">
import RefreshButton from '@/components/generic/RefreshButton.vue';
import LoadFileList from '@/components/global/upload_tabs/LoadFileList.vue';
import RouteMixin from '@/mixins/RouteMixin';
import ServerMixin from '@/mixins/ServerMixin';
import {EvaluationModule} from '@/store/evaluations';
import {FileID} from '@/store/report_intake';
import {IEvaluation} from '@heimdall/interfaces';
import Component, {mixins} from 'vue-class-component';
import {Prop, Watch} from 'vue-property-decorator';

/**
 * Uploads data to the store with unique IDs asynchronously as soon as data is entered.
 * Emits "got-files" with a list of the unique_ids of the loaded files.
 */
@Component({
  components: {
    LoadFileList,
    RefreshButton
  }
})
export default class DatabaseReader extends mixins(ServerMixin, RouteMixin) {
  @Prop({default: false}) readonly refresh!: boolean;

  headers: Object[] = [
    {
      text: 'Filename',
      align: 'start',
      sortable: true,
      value: 'filename'
    },
    {
      text: 'Groups',
      value: 'groups',
      sortable: true
    },
    {
      text: 'Tags',
      value: 'evaluationTags',
      sortable: true
    },
    {text: 'Uploaded', value: 'createdAt', sortable: true},
    {
      text: 'Actions',
      value: 'actions',
      align: 'right'
    }
  ];

  @Watch('refresh')
  onChildChanged(newRefreshValue: boolean, _oldValue: boolean) {
    if (newRefreshValue === true) {
      // Whenever refresh is set to true, call refresh on the database results
      this.get_all_results();
    }
  }

  mounted() {
    this.get_all_results();
  }

  async get_all_results(): Promise<void> {
    EvaluationModule.getAllEvaluations();
  }

  get loading() {
    return EvaluationModule.loading;
  }

  get files() {
    return EvaluationModule.allEvaluations;
  }

  load_results(evaluations: IEvaluation[]): void {
    EvaluationModule.load_results(
      evaluations.map((evaluation) => evaluation.id)
    ).then((fileIds: (FileID | void)[]) => {
      this.$emit('got-files', fileIds.filter(Boolean));
    });
  }
}
</script>
