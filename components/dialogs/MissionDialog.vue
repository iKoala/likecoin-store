<template>
  <div class="mission-dialog">
    <base-dialog
      ref="dialog"
      :class="[
        {
          upcoming: isUpcomingMission,
          'with-icon': !isUnfinishedAndExpired,
        }
      ]"
      :isShowCloseButton="true"
      @update:isShow="onDialogUpdate($event)">
      <div
        v-if="!isUnfinishedAndExpired"
        slot="header-center"
        class="lc-section-header-icon lc-dialog-icon lc-raised-icon">
        <mission-icon :mission-id="missionId" />
      </div>

      <transition name="lc-transition-default" mode="out-in">
        <div
          v-if="!shouldShowDesktopOnly && !isCompleted && isCustomLayout"
          key="custom"
          class="mission-dialog-content">

          <quote-tweet-mission
            v-if="missionId === 'twitter'"
            :userId="getUserInfo.user"
            @cancel="onDismiss"
            @complete="onCompleteMission" />

          <retweet-mission
            v-else-if="missionId === 'twitterBitmart'"
            :user-id="getUserInfo.user"
            :mission-id="missionId"
            tweet-url="https://twitter.com/BitMartExchange/status/1010271556742807552"
            @cancel="onDismiss"
            @complete="onCompleteMission" />

        </div>
        <div
          v-else
          key="generic"
          class="mission-dialog-content">

          <div class="lc-dialog-container-1">
            <div v-if="reward" class="reward-label">
              {{ reward }}
            </div>

            <h1 class="title-label">{{ title }}</h1>

            <div v-if="isUnfinishedAndExpired">
              <i18n
                class="description"
                path="Mission.notFound.description"
                tag="div"
              >
                <div place="bonusPage" @click="onDismiss">
                  <nuxt-link
                    class="lc-font-weight-600 lc-color-like-green lc-underline"
                    :to="{ name: 'in-bonus' }"
                  >{{
                    $t('Mission.notFound.bonusPage')
                  }}</nuxt-link>
                </div>
              </i18n>
            </div>
            <div v-else-if="isMissionRequired">
              <i18n
                class="description"
                :path="`Mission.missionRequired.${mission.require[0]}.description`"
                tag="div"
              >
                <span
                  class="lc-font-weight-600 lc-color-like-dark-brown-2"
                  place="bonusTask"
                >{{
                  $t(`Mission.${mission.id}.title`)
                }}</span>
              </i18n>
            </div>
            <div class="description" v-else-if="description" v-html="description" />

            <div class="description" v-if="subDescription" v-html="subDescription" />
          </div>

          <mission-completed-banner
            v-if="isCompleted || mission.isClaimed"
            class="lc-margin-top-32 lc-mobile"
            :animated="isCompleted"
            :isClaimed="mission.isClaimed"
            @click="onDismiss" />

          <div v-else-if="shouldShowDesktopOnly" class="lc-dialog-container-1">
            <simple-svg
              class="lc-color-like-green lc-margin-vertical-16"
              :filepath="RequiredDesktopIcon"
              width="290px"
              height="270px"
              fill="currentColor"
              stroke="transparent" />

            <p class="lc-color-like-gray-4 lc-color-like-gray-4 lc-margin-vertical-16">
              {{ $t('Mission.common.label.pleaseUseDesktop') }}
            </p>

            <div class="lc-text-align-center">
              <a
                class="lc-font-size-16 lc-underline"
                href="https://help.like.co/likecoin-faq/newbies/accessing-my-likecoin-id-on-a-computer-which-is-previously-registered-on-mobile" target="_blank">
                {{ $t('Mission.common.button.desktopFAQ') }}
              </a>
            </div>

            <div class="lc-button-group lc-margin-top-16">
              <md-button class="md-likecoin lc-cancel" @click="onDismiss">
              {{ $t('General.button.cancel') }}
              </md-button>
            </div>
          </div>

          <div
            v-else
            class="lc-dialog-container-1 lc-margin-top-24 lc-mobile">

            <!-- BEGIN - Mission Not Found Section -->
            <div
              v-if="mission.isExpired">

              <div class="lc-button-group">
                <md-button class="md-likecoin" @click="onDismiss">
                {{ $t('General.button.ok') }}
                </md-button>
              </div>

            </div>
            <!-- END - Mission Not Found Section -->

            <!-- BEGIN - Required Other Mission Section -->
            <div
              v-else-if="isMissionRequired">

              <div class="lc-button-group">
                <md-button class="md-likecoin" @click="onClickNextMission">
                  {{ $t('General.button.next') }}
                </md-button>
              </div>

            </div>
            <!-- END - Required Other Mission Section -->

            <!-- BEGIN - Getting Start Section -->
            <div
              v-else-if="mission.id === 'gettingStart'"
              class="getting-start-form">

              <task-list
                :tasks="getTasks"
                @click="onClickGettingStartTask" />

              <div class="lc-button-group">
                <md-button class="md-likecoin" @click="onDismiss">
                {{ $t('General.button.ok') }}
                </md-button>
              </div>

            </div>
            <!-- END - Getting Start Section -->

            <!-- BEGIN - Verify Email Section -->
            <div
              v-else-if="!isReferral && missionId === 'verifyEmail'"
              class="verify-email-form">

              <verify-email-form
                ref="form"
                :email="this.getUserInfo.email"
                :label="$t('Dialog.emailInput.label')"
                @cancel="onDismiss"
                @submit="onVerifyEmail"/>

            </div>
            <!-- END - Verify Email Section -->

            <!-- BEGIN - Invitee Verify Email Section -->
            <div
              v-else-if="isReferral && missionId === 'verifyEmail'"
              class="verify-email-form">

              <div class="lc-button-group">
                <md-button
                  class="md-likecoin"
                  @click="onDismiss">
                  {{ $t('General.button.ok') }}
                </md-button>
              </div>

            </div>
            <!-- END - Verify Email Section -->

            <!-- BEGIN - Invite Friend Section -->
            <invite-friend-form
              v-else-if="mission.id === 'inviteFriend'"
              class="lc-margin-top-24"
              form-id="mission-invite-friend-form"
              @invite="onInviteFriend" />
            <!-- END - Invite Friend Section -->

            <!-- BEGIN - Join Token Sale Section -->
            <div
              v-else-if="!isReferral && isJoinTokenSaleMission"
              class="join-tokensale-form">

              <div class="lc-button-group">
                <md-button
                  class="md-likecoin"
                  @click="$router.push({ name: 'in-tokensale' })"
                >
                  {{ $t(`Home.Sale.button.${this.isUpcomingMission
                      ? 'prepareToJoin' : 'joinNow'}`) }}
                </md-button>
                <br />
                <md-button
                  class="md-likecoin lc-cancel"
                  @click="onDismiss">
                  {{ $t('General.button.cancel') }}
                </md-button>
              </div>

            </div>
            <!-- END - Join Token Sale Section -->

            <!-- BEGIN - Invitee Join Token Sale Section -->
            <div
              v-else-if="isReferral && isJoinTokenSaleMission"
              class="join-tokensale-form">

              <div class="lc-button-group">
                <md-button
                  class="md-likecoin"
                  @click="onDismiss">
                  {{ $t('General.button.ok') }}
                </md-button>
              </div>

            </div>
            <!-- END - Join Token Sale Section -->

            <!-- BEGIN - Invite Token Sale Section -->
            <div
              v-else-if="missionId === 'inviteTokenSale'"
              class="invite-tokensale-form">

              <div class="lc-button-group">
                <md-button
                  class="md-likecoin"
                  @click="onDismiss">
                  {{ $t('General.button.ok') }}
                </md-button>
              </div>

            </div>
            <!-- END - Invite Token Sale Section -->
          </div>

        </div>
      </transition>

    </base-dialog>
  </div>
</template>


<script>
import { mapActions, mapGetters } from 'vuex';
import moment from 'moment';

import BaseDialog from '~/components/dialogs/BaseDialog';
import InviteFriendForm from '~/components/forms/InviteFriendForm';
import MissionCompletedBanner from '~/components/Mission/CompletedBanner';
import MissionIcon from '~/components/Mission/Icon';
import QuoteTweetMission from '~/components/dialogs/MissionDialogContent/QuoteTweet';
import RetweetMission from '~/components/dialogs/MissionDialogContent/Retweet';
import TaskList from '~/components/Mission/TaskList';
import VerifyEmailForm from '~/components/forms/VerifyEmailForm';
import { GETTING_STARTED_TASKS } from '@/constant';

import { logTrackerEvent } from '@/util/EventLogger';
import { openURL, checkIsMobileClient } from '@/util/client';

import LinkIcon from '@/assets/icons/fillable/link.svg';
import FacebookIcon from '@/assets/icons/fillable/facebook.svg';
import TwitterIcon from '@/assets/icons/fillable/twitter.svg';
import RequiredDesktopIcon from '@/assets/mission/misc/required-desktop.svg';

function timeout(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

export default {
  name: 'mission-dialog',
  components: {
    BaseDialog,
    InviteFriendForm,
    MissionCompletedBanner,
    MissionIcon,
    QuoteTweetMission,
    RetweetMission,
    TaskList,
    VerifyEmailForm,
  },
  data() {
    return {
      GETTING_STARTED_TASKS,
      FacebookIcon,
      LinkIcon,
      TwitterIcon,
      RequiredDesktopIcon,
      isReferral: false,
      isCompleted: false,
      invitee: '',
    };
  },
  computed: {
    ...mapGetters([
      'getPopupMission',
      'getMissionById',
      'getUserInfo',
      'getMissionList',
      'getSelectedMission',
    ]),
    mission() {
      return (
        this.getMissionById(this.getPopupMission.id) ||
        this.getSelectedMission ||
        {}
      );
    },
    isCustomLayout() {
      return /^twitter.*/.test(this.missionId);
    },
    reward() {
      if (this.mission.isFromUrl) return '';
      return this.mission.isReferral ? this.mission.referralReward : this.mission.reward;
    },
    title() {
      if (this.isUnfinishedAndExpired) {
        return this.$t('Mission.notFound.title');
      } else if (this.isMissionRequired) {
        return this.$t(`Mission.missionRequired.${this.mission.require[0]}.title`);
      }
      return this.$t(`Mission.${this.missionId}.title`);
    },
    description() {
      if (this.mission.isFromUrl) {
        return '';
      }
      if (this.shouldShowDesktopOnly) {
        return this.$t('Mission.common.label.desktopRequired');
      }

      const referralPostfix = this.isReferral ? 'Referral' : '';
      const { invitee } = this;
      return `
        ${this.mission.upcoming ? this.$t(
    'Mission.common.label.upcomingDate',
    { upcoming: moment.utc(this.mission.upcoming).local().format('D-M-YYYY') },
  )
    : ''}
        ${this.$t(`Mission.${this.missionId}${referralPostfix}.description`, { invitee })}
      `;
    },
    subDescription() {
      if (this.mission.isExpired) return null;
      if (this.hasReferrer && this.missionId === 'joinTokenSale') {
        return this.$t('Mission.joinTokenSale.subDescription');
      }
      return null;
    },
    missionId() {
      return this.mission.id || this.getPopupMission.id;
    },
    shouldShowDesktopOnly() {
      return this.mission.isDesktopOnly && checkIsMobileClient();
    },
    isJoinTokenSaleMission() {
      return (this.missionId === 'joinTokenSale' || this.missionId === 'refereeTokenSale');
    },
    getTasks() {
      return GETTING_STARTED_TASKS.map(id => ({
        id,
        title: this.$t(`Mission.${this.missionId}.${id}`),
        state: this.mission[id] ? 'completed' : 'active',
      }));
    },
    hasReferrer() {
      return this.getUserInfo.referrer;
    },
    isUpcomingMission() {
      return this.mission.upcoming;
    },
    isUnfinishedAndExpired() {
      return this.mission.isExpired && !this.mission.isClaimed;
    },
    isMissionRequired() {
      return this.mission.isMissionRequired && this.mission.require.length > 0;
    },
  },
  methods: {
    ...mapActions([
      'postStepMission',
      'refreshMissionList',
      'onMissionClick',
      'setPromptNotificationDialog',
    ]),
    show() {
      this.$refs.dialog.show();
    },
    hide() {
      this.$refs.dialog.hide();
    },
    refreshMissions() {
      this.refreshMissionList(this.getUserInfo.user);
    },
    onVerifyEmail() {
      // TODO
      this.hide();
    },
    onInviteFriend(type) {
      switch (type) {
        case 'email':
          logTrackerEvent(this, 'Mission', 'ClickGetFreeLikeCoin', 'email invite', 1);
          break;
        case 'url':
          logTrackerEvent(this, 'Mission', 'sendInvitation', 'copy invite link', 1);
          break;
        case 'facebook':
          logTrackerEvent(this, 'Mission', 'sendInvitation', 'fb share invite', 1);
          break;
        case 'twitter':
          logTrackerEvent(this, 'Mission', 'sendInvitation', 'twitter invite', 1);
          break;
        default:
      }
    },
    async onClickGettingStartTask(t) {
      /* post first due to trust problem */
      await Promise.race([
        this.postStepMission({
          user: this.getUserInfo.user,
          missionId: this.missionId,
          taskId: t.id,
        }),
        // magic time to allow popup and xhr call without being blocked
        timeout(23),
      ]);

      switch (t.id) {
        case 'taskPaymentPage':
          openURL(this, `/${this.getUserInfo.user}`, 'payment-page');
          break;

        case 'taskOnepager': {
          openURL(this, '/in/whitepaper', 'onpager-page');
          break;
        }

        case 'taskVideo': {
          let link = 'https://youtu.be/';
          switch (this.$i18n.locale) {
            case 'cn':
              link += '344nFHa7fC0';
              break;
            case 'zh':
              if (window.navigator.language === 'zh-HK') {
                link += 'MCeXL2WpK10';
              } else {
                link += '344nFHa7fC0';
              }
              break;
            case 'ja':
              link += 'YbMTQ3F1isU';
              break;
            case 'ru':
              link += 'MXkcdOg3Hkk';
              break;
            default:
              link += '28spMOgMs3o';
          }
          openURL(this, link, 'youtube');
          break;
        }

        case 'taskSocial': {
          let link = 'https://twitter.com/likecoin_fdn';
          switch (this.$i18n.locale) {
            case 'zh':
              link = 'https://www.facebook.com/LikeCoin.Foundation';
              break;
            default:
          }
          openURL(this, link, 'social-group');
          break;
        }
        default:
      }
    },
    async onCompleteMission() {
      this.hide();
      await this.onMissionClick({
        ...this.mission,
        done: true,
      });
      this.refreshMissions();
    },
    onDismiss() {
      this.hide();
      const isMissionDialogShow = false;
      this.onDialogUpdate(isMissionDialogShow);
    },
    onDialogUpdate(isMissionDialogShow) {
      if (this.getUserInfo.isEmailEnabled === false && !isMissionDialogShow && this.isCompleted) {
        this.setPromptNotificationDialog(true);
      }
    },
    onClickNextMission() {
      const mission = this.getMissionList.find(m => m.id === this.mission.require[0]);
      if (mission) {
        this.onMissionClick({
          ...mission,
        });
      } else {
        this.hide();
      }
    },
  },
  watch: {
    getPopupMission(m) {
      if (m) {
        const { invitee, isReferral, isCompleted } = m;
        this.invitee = invitee;
        this.isReferral = isReferral;
        this.isCompleted = !!isCompleted;
        this.show();
      }
    },
  },
};
</script>


<style lang="scss" scoped>
@import "~assets/variables";

.lc-dialog-icon {
  .upcoming & {
    background: #e6e6e6;
  }

  @media (max-width: 600px) {
    padding: 2px;
  }

  .mission-icon {
    color: $like-green;

    .upcoming & {
      color: #9b9b9b;
    }
  }
}

.title-label {
  color: $like-dark-brown-2;

  font-size: 32px;
  font-weight: 300;

  &:not(:last-child){
    margin-bottom: 8px;
  }
}

.reward-label {
  text-align: center;
  white-space: nowrap;
  text-overflow: ellipsis;

  color: $like-green;

  font-size: 20px;
  font-weight: 600;

  &:not(:last-child) {
    margin-bottom: 12px;
  }

  .small {
    font-size: 12px;
  }

  .upcoming & {
    color: $like-gray-4;
  }
}

.description {
  color: $like-gray-4;

  font-size: 16px;

  &:not(:last-child) {
    margin-bottom: 32px;
  }

  > div:not([place='']) {
    display: inline;
  }
}

.md-dialog.upcoming {
  :global(.lc-dialog-header::before) {
    background-image: linear-gradient(266deg, #ececec, #c0c0c0);
  }
}
</style>
