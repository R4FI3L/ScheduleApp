<template>
  <q-page class="bg-grey-3 column">
    <q-list class="bg-white" separator bordered>
      <q-item
        v-for="(task, index) in tasks"
        :key="index"
        @click="task.done = !task.done"
        clickable
        v-ripple
        :class="{ 'done bg-blue-1': task.done }"
      >
        <q-item-section avatar>
          <q-checkbox v-model="task.done" color="primary" class="no-pointer-events" />
        </q-item-section>
        <q-item-section>
          <q-item-label>{{task.title}}</q-item-label>
          <q-item-label caption>{{task.description}}</q-item-label>
        </q-item-section>
        <q-item-section side>
          <q-item-label>{{task.startTime}} - {{task.finishTime}}</q-item-label>
        </q-item-section>
        <q-item-section v-if="task.done" side>
          <q-btn @click.stop="editTask(index)" dense flat round color="primary" icon="edit" />
          <q-btn @click.stop="deleteTask(index)" dense flat round color="primary" icon="delete" />
        </q-item-section>
      </q-item>
    </q-list>
    <div v-if="!tasks.length" class="no-tasks absolute-center">
      <q-icon name="event" size="100px" color="primary" />
      <div class="text-h5 text-primary text-center">No hay</div>
    </div>
    <!---Dialog-->

    <q-dialog v-model="dialog" persistent>
      <q-card class="my-card">
        <q-card-section>Agregar un horario</q-card-section>

        <q-separator />

        <q-card-section class="q-pt-none">
          <q-input v-model="taskTitle" label="Titulo" />
        </q-card-section>

        <q-card-section class="q-pt-none">
          <q-input v-model="taskDescription" label="Descripcion" />
        </q-card-section>

        <q-card-section class="q-pt-none">
          Start:
          <q-input filled v-model="taskStartTime" mask="time" :rules="['taskStartTime']">
            <template v-slot:append>
              <q-icon name="access_time" class="cursor-pointer">
                <q-popup-proxy transition-show="scale" transition-hide="scale">
                  <q-time v-model="taskStartTime">
                    <div class="row items-center justify-end">
                      <q-btn v-close-popup label="Save" color="primary" flat />
                    </div>
                  </q-time>
                </q-popup-proxy>
              </q-icon>
            </template>
          </q-input>End:
          <q-input filled v-model="taskFinishTime" mask="time" :rules="['taskFinishTime']">
            <template v-slot:append>
              <q-icon name="access_time" class="cursor-pointer">
                <q-popup-proxy transition-show="scale" transition-hide="scale">
                  <q-time v-model="taskFinishTime">
                    <div class="row items-center justify-end">
                      <q-btn v-close-popup label="Save" color="primary" flat />
                    </div>
                  </q-time>
                </q-popup-proxy>
              </q-icon>
            </template>
          </q-input>
        </q-card-section>

        <q-separator />

        <q-card-actions align="right">
          <q-btn v-close-popup flat color="primary" @click="dialog = !dialog">Cancelar</q-btn>
          <q-btn v-close-popup flat color="primary" @click="addTask()">Agregar</q-btn>
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!--EDIT DIALOG-->

    <q-dialog v-model="editdialog" persistent>
      <q-card class="my-card">
        <q-card-section>Agregar un horario</q-card-section>

        <q-separator />

        <q-card-section class="q-pt-none">
          <q-input v-model="tempTask.title" label="Titulo" />
        </q-card-section>

        <q-card-section class="q-pt-none">
          <q-input v-model="tempTask.description" label="Descripcion" />
        </q-card-section>

        <q-card-section class="q-pt-none">
          Start:
          <q-input filled v-model="tempTask.startTime" mask="time" :rules="['taskStartTime']">
            <template v-slot:append>
              <q-icon name="access_time" class="cursor-pointer">
                <q-popup-proxy transition-show="scale" transition-hide="scale">
                  <q-time v-model="tempTask.startTime">
                    <div class="row items-center justify-end">
                      <q-btn v-close-popup label="Save" color="primary" flat />
                    </div>
                  </q-time>
                </q-popup-proxy>
              </q-icon>
            </template>
          </q-input>End:
          <q-input filled v-model="tempTask.finishTime" mask="time" :rules="['taskFinishTime']">
            <template v-slot:append>
              <q-icon name="access_time" class="cursor-pointer">
                <q-popup-proxy transition-show="scale" transition-hide="scale">
                  <q-time v-model="tempTask.finishTime">
                    <div class="row items-center justify-end">
                      <q-btn v-close-popup label="Save" color="primary" flat />
                    </div>
                  </q-time>
                </q-popup-proxy>
              </q-icon>
            </template>
          </q-input>
        </q-card-section>

        <q-separator />

        <q-card-actions align="right">
          <q-btn v-close-popup flat color="primary" @click="editdialog = !editdialog">Cancelar</q-btn>
          <q-btn v-close-popup flat color="primary">Guardar</q-btn>
        </q-card-actions>
      </q-card>
    </q-dialog>
    <!--EDIT DIALOG END-->
    <q-page-sticky position="bottom-right" :offset="[18, 18]">
      <q-btn fab icon="add" color="primary" @click="dialog = !dialog" />
    </q-page-sticky>
  </q-page>
</template>

<script>
export default {
  mounted: function () {
    let tareasDB = JSON.parse(localStorage.getItem("tareas-local"));
    if (tareasDB === null) {
      this.tasks = [];
      console.log("creado");
    } else {
      this.tasks = tareasDB;
    }
  },
  data() {
    return {
      taskTitle: "",
      taskStartTime: "00:00",
      taskFinishTime: "00:00",
      taskDescription: "",
      tempTask: "",
      dialog: false,
      editdialog: false,
      tasks: [],
    };
  },
  methods: {
    deleteTask(index) {
      this.$q
        .dialog({
          title: "Confirm",
          message: "Really Delete?",
          cancel: true,
          persistent: true,
        })
        .onOk(() => {
          this.tasks.splice(index, 1);
          localStorage.setItem("tareas-local", JSON.stringify(this.tasks));
          this.$q.notify("Task deleted");
        });
    },
    addTask() {
      if (this.newTask != "") {
        this.tasks.push({
          title: this.taskTitle,
          startTime: this.taskStartTime,
          finishTime: this.taskFinishTime,
          description: this.taskDescription,
          done: false,
        });
        this.taskTitle = "";
        this.taskDescription = "";
        this.taskStartTime = "00:00";
        this.taskFinishTime = "00:00";
        localStorage.setItem("tareas-local", JSON.stringify(this.tasks));
      }
    },
    editTask(index) {
      this.tempTask = this.tasks[index];
      console.log(this.tempTask);
      this.editdialog = true;
    },
  },
};
</script>

<style lang="scss">
.done {
  .q-item__label {
    text-decoration: line-through;
    color: #bbb;
  }
}
.no-tasks {
  opacity: 0.5;
}
</style>