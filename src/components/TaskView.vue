<template>
  <div class="container">
    <nav
      class="navbar shadow bg-white rounded justify-content-between flex-nowrap flex-row fixed-top"
    >
      <div class="container">
        <a class="navbar-brand float-left" href="/tasks">
          SNK-Y-BOT
        </a>

        <ul class="nav navbar-nav flex-row float-right">
          <li class="nav-item">
            <router-link class="nav-link pr-3" to="/tasks">Tasks</router-link>
          </li>
          <li class="nav-item">
            <router-link class="nav-link pr-3" to="/billing"
              >Billing</router-link
            >
          </li>
          <li class="nav-item">
            <router-link class="nav-link pr-3" to="/leaderboard"
              >Leaderboard</router-link
            >
          </li>
          <li class="nav-item">
            <router-link  class="nav-link pr-3" to="/"
              >Logout</router-link
            >
          </li>
        </ul>
      </div>
    </nav>
    <div class="table-wrapper">
      <div class="row">
        <div class="col-75">
          <div class="card-container">
            <div class="row">
              <div class="col-50">
                <h3>Add New Task</h3>

                <label for="fname"> Website </label>
                <select v-model="website">
                  <option
                    v-for="(sites, pos) in sitelist"
                    :value="sites"
                    :key="pos"
                  >
                    {{ sites }}
                  </option>
                </select>

                <label for="email"><i class="fa fa-user" /> Profile </label>
                <select class="dropdown" v-model="selectedProfile">
                  <option
                    v-for="(profs, pos) in billingProfiles"
                    :value="profs"
                    :key="pos"
                  >
                    {{ profs.profile }}
                  </option>
                </select>
                <br />

                <label for="adr"> ATC Link </label>
                <input type="text" id="adr" v-model="atcLink" />

                <label for="adr"> Size </label>
                <input type="text" id="adr" v-model="selectedsize" />

                <label> Product </label>
                <input type="text" v-model="selectedproduct" />
              </div>
            </div>

            <input
              type="submit"
              value="Add Task"
              class="save-btn"
              @click="createTask"
            />
            {{ message }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { FirebaseFirestore } from "@firebase/firestore-types";
import { FirebaseAuth } from "@firebase/auth-types";

@Component
export default class TaskView extends Vue {
  readonly $appDB!: FirebaseFirestore;
  readonly $appAuth!: FirebaseAuth;
  private message = "";
  private uid = "none";
  private billingProfiles: any[] = [];
  private selectedProfile = {};
  private sitelist: any[] = [
    "Kith",
    "Dead Stock",
    "Shoe Palace",
    "Shop Nice Kicks",
  ];
  private website = "";
  private selectedsize = "";
  private atcLink = "";
  private selectedproduct = "";
  showMessage(m: string): void {
    this.message = m;
    setTimeout(() => {
      // Auto disappear after 5 seconds
      this.message = "";
    }, 5000);
  }
  makeid(): string {
    var result = [];
    var characters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
    var charactersLength = characters.length;
    for (var i = 0; i < 5; i++) {
      result.push(
        characters.charAt(Math.floor(Math.random() * charactersLength))
      );
    }
    return result.join("");
  }
  createTask(): void {
    this.uid = this.$appAuth.currentUser?.uid ?? "none";
    var TID: string = this.makeid();
    var task = {
      tid: TID,
      site: this.website,
      profile: this.selectedProfile,
      size: this.selectedsize,
      item: this.selectedproduct,
      atc: this.atcLink,
      status: "idle...",
    };
    this.$appDB
      .collection(`users/${this.uid}/tasks`)
      .add(task)
      .then(() => {
        this.$router.push({ path: "/tasks" });
      })
      .then(() => {
        console.log("Task: ", task);
        this.showMessage(`Task added successfully!`);
      });
  }
  mounted() {
    this.uid = this.$appAuth.currentUser?.uid ?? "none";
    console.log("THIS UID:", this.uid)
    this.$appDB.collection(`users/${this.uid}/billing`).onSnapshot((qs) => {
      this.billingProfiles.splice(0);
      qs.forEach((qds) => {
        if (qds.exists) {
          const billingInfo = qds.data();
          this.billingProfiles.push({
            name: billingInfo.name,
            email: billingInfo.email,
            address: billingInfo.address,
            city: billingInfo.city,
            state: billingInfo.state,
            zip: billingInfo.zip,
            cardName: billingInfo.cardName,
            cardNumber: billingInfo.cardNumber,
            expMonth: billingInfo.expMonth,
            expYear: billingInfo.expYear,
            cvv: billingInfo.cvv,
            profile: billingInfo.profile,
          });
        }
      });
    });
    console.log("logs: ", this.billingProfiles);
  }
}
</script>

<style scope>
.table-wrapper {
  margin: auto;
  margin-top: 10%;
  margin-left: 15%;
  width: 100%;
  background: #ffffff;
  box-shadow: 0px 14px 80px rgba(34, 35, 58, 0.2);
  padding: 40px 55px 45px 55px;
  border-radius: 15px;
  transition: all 0.3s;
}

.row {
  display: -ms-flexbox; /* IE10 */
  display: flex;
  -ms-flex-wrap: wrap; /* IE10 */
  flex-wrap: wrap;
  margin: 0 -16px;
}

.col-25 {
  -ms-flex: 25%; /* IE10 */
  flex: 25%;
}

.col-50 {
  -ms-flex: 50%; /* IE10 */
  flex: 50%;
}

.col-75 {
  -ms-flex: 75%; /* IE10 */
  flex: 75%;
}

.col-25,
.col-50,
.col-75 {
  padding: 0 16px;
}

.card-container {
  background-color: #f2f2f2;
  padding: 5px 20px 15px 20px;
  border: 1px solid lightgrey;
  border-radius: 3px;
}

input[type="text"] {
  width: 100%;
  margin-bottom: 20px;
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 3px;
}

label {
  margin-bottom: 10px;
  display: block;
}

.icon-container {
  margin-bottom: 20px;
  padding: 7px 0;
  font-size: 24px;
}

.save-btn {
  background-color: #6441a5;
  color: white;
  padding: 12px;
  margin: 10px 0;
  border: none;
  width: 100%;
  border-radius: 3px;
  cursor: pointer;
  font-size: 17px;
}

.save-btn:hover {
  background-color: #493077;
}

span.price {
  float: right;
  color: grey;
}
</style>
