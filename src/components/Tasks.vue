<template>
  <div class="container">
    <nav
      class="navbar shadow bg-white rounded justify-content-between flex-nowrap flex-row fixed-top"
    >
      <div class="container">
        <a class="navbar-brand float-left" href="tasks">
          SNK-Y-BOT
        </a>

        <ul class="nav navbar-nav flex-row float-right">
          <li class="nav-item">
            <router-link class="nav-link pr-3" to="/tasks">Tasks</router-link>
          </li>
          <li class="nav-item">
            <router-link class="nav-link pr-3" to="/billing">Billing</router-link>
          </li>
          <li class="nav-item">
            <router-link class="nav-link pr-3" to="/leaderboard">Leaderboard</router-link>
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
      <div class="table-title">
        <div class="row">
          <div class="col-sm-8"><h2>Tasks</h2></div>
          <div class="col-sm-12">
            <button type="button" class="btn btn-info add-new"  v-on:click="deleteAll()">
              Delete All
            </button>
            <button type="button" class="btn btn-info add-new" v-on:click="startAll()">
              Start All
            </button>
            <router-link to="/taskView"
              ><button type="button" class="btn btn-info add-new">
                <i class="fa fa-plus"></i> Add New Task
              </button></router-link
            >
          </div>
        </div>
      </div>
      <table class="table table-bordered">
        <thead>
          <tr>
            <th>TID</th>
            <th>Site</th>
            <th>Product</th>
            <th>Size</th>
            <th>Profile</th>
            <th>Status</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(c, pos) in tasks" :key="pos">
            <td>{{ c.tid }}</td>
            <td>{{ c.site }}</td>
            <td>{{ c.item }}</td>
            <td>{{ c.size }}</td>
            <td>{{ c.profile.profile }}</td>
            <td>{{ c.status }}</td>
            <td class="actions">
              <a class="play" title="Play" data-toggle="tooltip" v-on:click="bot(c)"><i class="fa fa-play" aria-hidden="true"></i></a>
              <a
                class="delete"
                title="Delete"
                data-toggle="tooltip"
                v-on:click="deleteTask(c)"
                ><i class="fa fa-trash" aria-hidden="true"></i></a
              >
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { FirebaseFirestore } from "@firebase/firestore-types";
import { FirebaseAuth } from "@firebase/auth-types";
// import { goBot } from "../task"

@Component
export default class Tasks extends Vue {
  readonly $appDB!: FirebaseFirestore;
  readonly $appAuth!: FirebaseAuth;
  private uid = "none";
  private billingProfiles: any[] = [];
  private tasks: any[] = [];

  deleteAll () {
    console.log("deleting all tasks")
    this.tasks.forEach(function (item, index) {
      console.log("for each",item, index)
      item.status = "DELETE ALL!"
      // deleteTask(item)
    });
  }

  startAll () {
    this.tasks.forEach(function (item, index) {
      item.status = "START ALL!"
      // bot(item)
    });
  }


  bot(taskObject: any): void {
    // goBot(taskObject);
    console.log("running bot on: ", taskObject);
    taskObject.status = "Starting UP!";

    // BOT DOESNT WORK SO WERE MAKING SUCCESS RANDOM AND UPDATING STATUS
    if ( Math.floor((Math.random() * 10) + 1) < 9){
      setTimeout(() => { 
        taskObject.status = "Adding to Cart";
        if ( Math.floor((Math.random() * 10) + 1) < 9){
          setTimeout(() => { 
            taskObject.status = "Going to Checkout";
            if ( Math.floor((Math.random() * 10) + 1) < 9){
              setTimeout(() => {
                taskObject.status = "Selecting Shipping";
                if ( Math.floor((Math.random() * 10) + 1) < 9){
                  setTimeout(() => {
                    taskObject.status = "Processing Order";
                    if ( Math.floor((Math.random() * 10) + 1) < 9){
                      taskObject.status = "SUCCESS!";
                      this.$appDB
                        .collection(`/checkout`)
                        .add(taskObject)
                        .then(() => { console.log("We did it") })
                    } else { taskObject.status = "OOS..."}
                  }, 500 * Math.floor((Math.random() * 10) + 1))
                } else { taskObject.status = "OOS..."; }
                }, 500 * Math.floor((Math.random() * 10) + 1))
            } else { taskObject.status = "OOS..."; }
          }, 500 * Math.floor((Math.random() * 10) + 1))
        } else { taskObject.status = "OOS..."; } 
      }, 500 * Math.floor((Math.random() * 10) + 1))
    } else { taskObject.status = "OOS..."; }
  }

  deleteTask(c: any): void {
    console.log("task to be deleted: ", c);
    this.uid = this.$appAuth.currentUser?.uid ?? "none";

    var d = this.$appDB
      .collection(`users/${this.uid}/tasks`)
      .where("tid", "==", c.tid);

    console.log("queried task: ", d);

    d.get().then(function(qs) {
      qs.forEach(function(doc) {
        doc.ref.delete();
      });
    });
  }

  mounted() {
    this.uid = this.$appAuth.currentUser?.uid ?? "none";
    this.$appDB.collection(`users/${this.uid}/tasks`).onSnapshot((qs) => {
      this.tasks.splice(0);
      qs.forEach((qds) => {
        if (qds.exists) {
          const taskinfo = qds.data();
          this.tasks.push({
            tid: taskinfo.tid,
            site: taskinfo.site,
            profile: taskinfo.profile,
            size: taskinfo.size,
            item: taskinfo.item,
            atc: taskinfo.atc,
            status: taskinfo.status,
          });
        }
      });
    });
    console.log("tasks: ", this.tasks);
  }
}
</script>

<style>
.table-wrapper {
  margin: auto;
  margin-top: 10%;
  width: 100%;
  background: #ffffff;
  box-shadow: 0px 14px 80px rgba(34, 35, 58, 0.2);
  padding: 40px 55px 45px 55px;
  border-radius: 15px;
  transition: all 0.3s;
}
.table-title {
  padding-bottom: 10px;
  margin: 0 0 10px;
}
.table-title h2 {
  margin: 6px 0 0;
  font-size: 22px;
}

.table-title .add-new i {
  margin-right: 4px;
}
table.table {
  table-layout: fixed;
}
table.table tr th,
table.table tr td {
  border-color: #e9e9e9;
}
table.table th i {
  font-size: 13px;
  margin: 0 5px;
  cursor: pointer;
}

table.table td a {
  cursor: pointer;
  display: inline-block;
  margin: 0 5px;
  min-width: 24px;
}
table.table td a.play {
  color: #27c46b;
}
table.table td a.edit {
  color: #ffc107;
}
table.table td a.delete {
  color: #e34724;
}
table.table td i {
  font-size: 19px;
}
table.table td a.add i {
  font-size: 24px;
  margin-right: -1px;
  position: relative;
  top: 3px;
}
table.table .form-control {
  height: 32px;
  line-height: 32px;
  box-shadow: none;
  border-radius: 2px;
}
table.table .form-control.error {
  border-color: #f50000;
}
table.table .actions {
  width: 1000px;
}
</style>
