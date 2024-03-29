<template>
  <div class="settings" style="background-color: white; padding: 16px">
    <h1>Settings</h1>

    <div>
      <h2>Tag</h2>
      <tag-container />
    </div>

    <div v-if="false">
      <hr style="width: 80%; margin: 64px auto; border: 0.5px dashed lightgray" />

      <h2>Notification</h2>
      <div class="cleararea">
        <div class="cleararea__flex-parent">
          <div class="cleararea__flex-left">
            <p>
              <span class="inline__vertical-middle">Desktop notification</span>
            </p>
          </div>
          <div class="cleararea__flex-right" style="text-align: right">
            <span class="inline__vertical-middle" v-if="notificationPermission != 'granted'">
              <a @click="desktopNotificationRequest">Request</a>
            </span>
            <span v-else>
              <a @click="desktopNotificationTesting">Test</a>
            </span>
          </div>
        </div>
      </div>
    </div>

    <div>
      <hr style="width: 80%; margin: 64px auto; border: 0.5px dashed lightgray" />

      <div>
        <h2>Feature enabled</h2>

        <table style="width: 480px; text-align: center" class="stripe-table">
          <thead>
            <tr>
              <th>Name</th>
              <th>Status</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>Backlog</td>
              <td>
                <input type="checkbox" :checked="featureBacklogEnabled" @change="enableMutation($event, 'backlog')" />
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div>
      <hr style="width: 80%; margin: 64px auto; border: 0.5px dashed lightgray" />

      <h2>Data</h2>
      <ul>
        <li>
          <router-link :to="{ name: 'ArchiveTasksView' }">Archive tasks</router-link>
        </li>
      </ul>
      <div>
        <h3>Backup</h3>
        <div class="cleararea">
          <div class="cleararea__flex-parent">
            <div class="cleararea__flex-left">
              <p>
                <span class="inline__vertical-middle">Download backup data</span>
              </p>
            </div>
            <div class="cleararea__flex-right" style="text-align: right">
              <span class="inline__vertical-middle">
                <button class="action" @click="exportdata">Export</button>
              </span>
            </div>
          </div>
        </div>
        <div class="cleararea">
          <div class="cleararea__flex-parent">
            <div class="cleararea__flex-left">
              <p>
                <span class="inline__vertical-middle">Restoration from backup data</span>
              </p>
            </div>
            <div class="cleararea__flex-right" style="text-align: right">
              <span class="inline__vertical-middle">
                <label class="btn action color-default" for="backup-file-uploader" style="font-size: 13px; font-weight: 400">
                  <input type="file" style="display: none" id="backup-file-uploader" @change="importdata">Import
                </label>
              </span>
            </div>
          </div>
        </div>
      </div>

      <h3>Clear</h3>
      <div class="cleararea">
        <div class="cleararea__flex-parent">
          <div class="cleararea__flex-left">
            <p>
              <span class="inline__vertical-middle">Clear task data</span>
            </p>
          </div>
          <div class="cleararea__flex-right" style="text-align: right">
            <span class="inline__vertical-middle">
              <button class="action" @click="clearTaskData">Execute</button>
            </span>
          </div>
        </div>
      </div>
      <div class="cleararea">
        <div class="cleararea__flex-parent">
          <div class="cleararea__flex-left">
            <p>
              <span class="inline__vertical-middle">Clear all data</span>
            </p>
          </div>
          <div class="cleararea__flex-right" style="text-align: right">
            <span class="inline__vertical-middle">
              <button class="action" @click="clearAllData">Execute</button>
            </span>
          </div>
        </div>
      </div>
    </div>

    <hr style="width: 80%; margin: 64px auto; border: 0.5px dashed lightgray" />

    <div>
      <h2>Contact</h2>
      <table style="width: 60%">
        <tr>
          <th>Twitter</th>
          <td>
            <a href="https://twitter.com/nogissh" target="_blank">@nogissh</a>
          </td>
        </tr>
        <tr>
          <th>Bug report</th>
          <td>
            Twitter or <a href="https://github.com/nogissh/prisall/issues" target="_blank">GitHub Issues</a>
          </td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
import TagContainer from '@/components/TagContainer.vue';

export default {
  name: 'SettingsView',
  components: {
    TagContainer,
  },
  computed: {
    featureBacklogEnabled: {
      get () {
        return this.$store.getters['featureEnabled/backlog'];
      }
    }
  },
  data () {
    return {
      notificationPermission: window.Notification.permission,
    }
  },
  methods: {
    desktopNotificationRequest: function () {
      Notification.requestPermission(function (result) {
        switch (result) {
          case 'granted':
            alert('Thanks, desktop notification is enabled.');
            new Notification('Do you see desktop notification?');
            break;
          case 'denied':
            alert('If you want to use notification, you can be enable on setting page.');
            break;
          default:
            break;          
        }
      });
    },
    desktopNotificationTesting: function () {
      var n = new Notification('title', { body: 'This is the test.' });
      n.onclose = function () { alert('Desktop notification is enabled.'); }
    },
    enableMutation: function (e, featName) {
      this.$store.dispatch('featureEnabled/' + featName, e.target.checked);
    },
    exportdata: function () {
      let data = {
        task: this.$store.getters['task/list'],
        tag: this.$store.getters['tag/list'],
        archivetask: this.$store.getters['archivetask/list'],
        backlog: this.$store.getters['backlog/list'],
      }
      let blob = new Blob([JSON.stringify(data)], { type: 'application/json' })
      let link = document.createElement('a')
      link.href = window.URL.createObjectURL(blob)
      link.download = 'prisall_backup_' + String(Date.now()) + '.json'
      link.click()
    },
    importdata: function (event) {
      let reader = new FileReader()
      let file = event.target.files[0]
      let type = file.type
      if (type != 'application/json') {
        alert('Invalid data format.');
        event.currentTarget.value = ''
        return;
      }
      reader.onload = () => {
        let data = JSON.parse(reader.result);
        this.$store.dispatch('task/overwrite', data.task);
        this.$store.dispatch('tag/overwrite', data.tag);
        this.$store.dispatch('archivetask/overwrite', data.archivetask);
        this.$store.dispatch('backlog/overwrite', data.backlog);
        alert('Welcome back!!!');
      };
      reader.readAsText(file);
    },
    clearTaskData: function () {
      if (! confirm('This operation can not be undone.')) { return; }
      this.$store.dispatch('task/clearAll');
      alert('Task data cleared.')
    },
    clearAllData: function () {
      if (! confirm('This operation can not be undone.')) { return; }
      this.$store.dispatch('task/clearAll');
      this.$store.dispatch('tag/clearAll');
      this.$store.dispatch('archivetask/clear');
      this.$store.dispatch('backlog/clear');
      this.$store.dispatch('featureEnabled/reset');
      alert('All data cleared.')
    }
  }
}
</script>

<style>
.stripe-table {
  border-collapse: collapse;
}

.stripe-table tr:nth-child(odd) {
  background-color: #fff;
}

.stripe-table tr:nth-child(even) {
  background-color: #eee;
}

.stripe-table tr td {
  border: 0;
}
</style>
