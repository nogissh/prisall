<template>
  <div id="task-detail-modal__background" @click="backgroundClickAction">
    <div id="task-detail-modal__container">
      <div id="task-detail-modal__body">
        <div style="padding: 4px 32px;" @click.stop>
          <h2>Task</h2>
          <div style="width: 100%">
            <table style="width: 80%; min-width: 640px;" id="task-detail-table">
              <tr>
                <th>Title</th>
                <td>
                  <input type="text" placeholder="Title" v-model="task.name" @keydown.enter="submit" id="task-detail-modal-title-input" />
                </td>
              </tr>
              <tr>
                <th>Description</th>
                <td>
                  <textarea v-model="task.description" style="height: 80px"></textarea>
                </td>
              </tr>
              <tr>
                <th>URL</th>
                <td>
                  <input type="text" placeholder="URL" v-model="task.reference" @keydown.enter="submit" />
                </td>
              </tr>
              <tr>
                <th>Tag</th>
                <td v-if="tags.length > 0">
                  <span
                    v-for="(tag, index) in tags"
                    :key="index"
                    class="tag"
                    :title="tag.name"
                    v-bind:style="tasktagids.indexOf(tag.id) >= 0 ? { 'background-color': 'lightgray' } : { 'background-color': 'white' }"
                    style="margin-right: 8px; border: 1px solid lightgray; cursor: pointer"
                    @click="mutatetasktag(index)"
                    >{{ tag.name }}</span>
                </td>
                <td v-else>
                  No tag created.
                </td>
              </tr>
              <tr>
                <th>Done</th>
                <td>
                  <input type="checkbox" v-model="task.done" />
                </td>
              </tr>
            </table>
          </div>
        </div>
      </div>

      <div id="task-detail-modal__footer" @click.stop>
        <div style="margin-top: 12px; text-align: center">
          <button class="action secondary" @click="closeModal">Cancel</button>
          <button class="action primary" style="margin-left: 8px" @click="submit">Save</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TaskDetailModal',
  data () {
    return {
      task: null,
      tasktagids: [],
    }
  },
  computed: {
    tags: {
      get () {
        return this.$store.getters['tag/list'];
      }
    }
  },
  methods: {
    backgroundClickAction: function () {
      this.closeModal();
    },
    closeModal: function () {
      this.$store.commit('modal/taskDetailVisible', false);
    },
    mutatetasktag: function (idx) {
      if (this.tasktagids.indexOf(this.tags[idx].id) == -1) {
        this.task.tags.push(this.tags[idx]);
      } else {
        this.task.tags = JSON.parse(JSON.stringify(this.task.tags.filter(tag => tag.id != this.tags[idx].id)));
      }
      this.tasktagids = this.task.tags.map(tag => tag.id);
    },
    submit: function (e) {
      if (e.keyCode == 229) return;
      if (this.task.name == '') return;
      this.$store.dispatch('task/update', this.task);
      this.closeModal();
    }
  },
  beforeMount () {
    let task = JSON.parse(JSON.stringify(this.$store.getters['task/get']));
    this.task = task;
    this.tasktagids = task.tags.map(tag => tag.id);
  },
  mounted () {
    document.getElementById('task-detail-modal-title-input').focus();
  }
}
</script>

<style>
#task-detail-modal__background {
  z-index: 1000;
  position: fixed;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  background-color: rgba(100, 100, 100, 0.5);
}

#task-detail-modal__container {
  z-index: 1001;
  position: absolute;
  width: 860px;
  height: 540px;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin: auto;
  background-color: white;
  border-radius: 0;
}

#task-detail-modal__body {
  width: 100%;
  height: 480px;
  overflow-y: scroll;
}

#task-detail-modal__footer {
  width: 100%;
  height: 59px;
  border-top: 1px lightgray solid;
}

#task-detail-modal__close-button:hover {
  cursor: pointer;
}

#task-detail-table td {
  padding: 16px 0;
}
</style>
