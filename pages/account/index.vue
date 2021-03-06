<template>
  <div>
    <section>
      <b-container class="account-page main pt-4 pb-5">
        <template v-if="parsedAccount">
          <div class="row">
            <div class="col-12 text-center">
              <h4 class="mb-1">Account {{ parsedAccount.accountId }}</h4>
            </div>
          </div>
          <div class="card mt-4 mb-3">
            <div class="card-body">
              <p class="text-center mb-2">
                <Identicon
                  :key="parsedAccount.accountId"
                  :value="parsedAccount.accountId"
                  :size="80"
                  :theme="'polkadot'"
                />
              </p>
              <h4 class="text-center mb-4">
                {{ parsedAccount.accountId }}
              </h4>
              <h4
                v-b-tooltip.hover
                class="text-center mb-4 amount"
                title="Free Balance"
              >
                {{ formatAmount(parsedAccount.balances.freeBalance) }}
              </h4>
              <table class="table table-striped">
                <tbody>
                  <tr>
                    <td>Account ID</td>
                    <td class="text-right">
                      <a
                        :href="blockExplorer.account + parsedAccount.accountId"
                        target="_blank"
                        class="d-block my-2"
                      >
                        <Identicon
                          :key="parsedAccount.accountId"
                          :value="parsedAccount.accountId"
                          :size="20"
                          :theme="'polkadot'"
                        />
                        <span
                          v-b-tooltip.hover
                          title="See address in PolkaScan"
                          >{{ parsedAccount.accountId }}</span
                        >
                      </a>
                    </td>
                  </tr>
                  <tr v-if="parsedAccount.identity.display">
                    <td>Identity::display</td>
                    <td class="text-right">
                      {{ parsedAccount.identity.display }}
                    </td>
                  </tr>
                  <tr v-if="parsedAccount.identity.email">
                    <td>Identity::email</td>
                    <td class="text-right">
                      <a
                        :href="`mailto:${parsedAccount.identity.email}`"
                        target="_blank"
                        >{{ parsedAccount.identity.email }}</a
                      >
                    </td>
                  </tr>
                  <tr v-if="parsedAccount.identity.legal">
                    <td>Identity::legal</td>
                    <td class="text-right">
                      {{ parsedAccount.identity.legal }}
                    </td>
                  </tr>
                  <tr v-if="parsedAccount.identity.riot">
                    <td>Identity::riot</td>
                    <td class="text-right">
                      {{ parsedAccount.identity.riot }}
                    </td>
                  </tr>
                  <tr v-if="parsedAccount.identity.web">
                    <td>Identity::web</td>
                    <td class="text-right">
                      <a :href="parsedAccount.identity.web" target="_blank">{{
                        parsedAccount.identity.web
                      }}</a>
                    </td>
                  </tr>
                  <tr v-if="parsedAccount.identity.twitter">
                    <td>Identity::twitter</td>
                    <td class="text-right">
                      <a
                        :href="
                          `https://twitter.com/${parsedAccount.identity.twitter.substr(
                            1,
                            parsedAccount.identity.twitter.length
                          )}`
                        "
                        target="_blank"
                        >{{ parsedAccount.identity.twitter }}</a
                      >
                    </td>
                  </tr>
                  <tr v-if="parsedAccount.identity.judgements">
                    <td>Identity::judgements</td>
                    <td class="text-right">
                      <span v-if="parsedAccount.identity.judgements.length > 0">
                        {{ parsedAccount.identity.judgements }}
                      </span>
                      <span>
                        No
                      </span>
                    </td>
                  </tr>
                  <tr>
                    <td>Account Nonce</td>
                    <td class="text-right">
                      {{ parsedAccount.balances.accountNonce }}
                    </td>
                  </tr>
                  <tr>
                    <td>Total Balance</td>
                    <td class="text-right">
                      {{ formatAmount(parsedAccount.balances.freeBalance) }}
                    </td>
                  </tr>
                  <tr>
                    <td>Available Balance</td>
                    <td class="text-right">
                      {{
                        formatAmount(parsedAccount.balances.availableBalance)
                      }}
                    </td>
                  </tr>
                  <tr>
                    <td>Locked Balance</td>
                    <td class="text-right">
                      {{ formatAmount(parsedAccount.balances.lockedBalance) }}
                    </td>
                  </tr>
                  <tr>
                    <td>Reserved Balance</td>
                    <td class="text-right">
                      {{ formatAmount(parsedAccount.balances.reservedBalance) }}
                    </td>
                  </tr>
                  <tr>
                    <td>Is Vesting?</td>
                    <td class="text-right">
                      {{ parsedAccount.balances.isVesting ? `Yes` : `No` }}
                    </td>
                  </tr>
                  <tr>
                    <td>Vested Balance</td>
                    <td class="text-right">
                      {{ formatAmount(parsedAccount.balances.vestedBalance) }}
                    </td>
                  </tr>
                  <tr>
                    <td>Vesting Total</td>
                    <td class="text-right">
                      {{ formatAmount(parsedAccount.balances.vestingTotal) }}
                    </td>
                  </tr>
                  <tr>
                    <td>Voting Balance</td>
                    <td class="text-right">
                      {{ formatAmount(parsedAccount.balances.votingBalance) }}
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </template>
      </b-container>
    </section>
  </div>
</template>
<script>
import { mapMutations } from "vuex";
import Identicon from "../../components/identicon.vue";
import { blockExplorer } from "../../polkastats.config.js";
import commonMixin from "../../mixins/commonMixin.js";
import gql from "graphql-tag";

export default {
  components: {
    Identicon
  },
  mixins: [commonMixin],
  data: function() {
    return {
      accountId: this.$route.query.accountId,
      blockExplorer,
      parsedAccount: undefined
    };
  },
  watch: {
    $route() {
      this.accountId = this.$route.query.accountId;
    }
  },
  apollo: {
    account: {
      query: gql`
        query account($account_id: String!) {
          account(where: { account_id: { _eq: $account_id } }) {
            account_id
            balances
            block_height
            identity
            timestamp
          }
        }
      `,
      variables() {
        return {
          account_id: this.$route.query.accountId
        };
      },
      result({ data }) {
        this.parsedAccount = {
          accountId: data.account[0].account_id,
          balances: JSON.parse(data.account[0].balances),
          blockHeight: data.account[0].block_height,
          identity:
            data.account[0].identity !== ``
              ? JSON.parse(data.account[0].identity)
              : {},
          timestamp: data.account[0].timestamp
        };
      }
    }
  },
  head() {
    return {
      title: "PolkaStats - Kusama account " + this.$route.query.accountId,
      meta: [
        {
          hid: "description",
          name: "description",
          content: "Kusama account " + this.$route.query.accountId
        }
      ]
    };
  }
};
</script>
<style>
.identicon {
  cursor: pointer;
}
.account-page .table tr td:first-child {
  width: 30%;
  font-weight: bold;
}
.account-page .table tr td:nth-child(2) {
  width: 70%;
}
.account-page .table tr td .identicon {
  display: inline-block;
}
.account-page .amount {
  color: #ef1073;
  font-weight: 700;
}
</style>
