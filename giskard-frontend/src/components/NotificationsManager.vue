<template>
  <div>
    <v-snackbar v-model='show' :color='currentNotificationColor'>
      <v-progress-circular class='ma-2' indeterminate v-show='showProgress'></v-progress-circular>

      {{ currentNotificationContent }}

      <template v-slot:action='{ attrs }'>
        <v-btn
            text
            v-bind='attrs'
            @click.native='close'
        >
          <v-icon>close</v-icon>
        </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>
<script lang='ts'>
import {Component, Vue, Watch} from 'vue-property-decorator';
import {AppNotification} from '@/store/main/state';
import {commitRemoveNotification} from '@/store/main/mutations';
import {readFirstNotification} from '@/store/main/getters';
import {dispatchRemoveNotification} from '@/store/main/actions';

@Component
export default class NotificationsManager extends Vue {
  public show: boolean = false;
  public text: string = '';
  public showProgress: boolean = false;
  public currentNotification: AppNotification | false = false;

  public async hide() {
    this.show = false;
    await new Promise((resolve) => setTimeout(() => resolve(undefined), 500));
  }

  public async close() {
    await this.hide();
    await this.removeCurrentNotification();
  }

  public async removeCurrentNotification() {
    if (this.currentNotification) {
      commitRemoveNotification(this.$store, this.currentNotification);
    }
  }

  public get firstNotification() {
    return readFirstNotification(this.$store);
  }

  public async setNotification(notification: AppNotification | false) {
    if (this.show) {
      await this.hide();
    }
    if (notification) {
      this.currentNotification = notification;
      this.showProgress = notification.showProgress || false;
      this.show = true;
    } else {
      this.currentNotification = false;
    }
  }

  @Watch('firstNotification')
  public async onNotificationChange(
      newNotification: AppNotification | false,
  ) {
    if (newNotification !== this.currentNotification) {
      await this.setNotification(newNotification);
      if (newNotification) {
        dispatchRemoveNotification(this.$store, {notification: newNotification, timeout: 2500});
      }
    }
  }

  public get currentNotificationContent() {
    return this.currentNotification && this.currentNotification.content || '';
  }

  public get currentNotificationColor() {
    return this.currentNotification && this.currentNotification.color || 'info';
  }
}
</script>
