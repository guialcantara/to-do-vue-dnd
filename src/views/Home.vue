<template>
  <v-container>
    <div class="d-flex flex-column flex-md-row justify-space-between">
      <v-col
        cols="12"
        md="3"
        v-for="(list, index) in lists"
        :key="index"
        class="list--area d-flex flex-column"
      >
        <h2>{{ list.title }}</h2>
        <v-card class="list rounded-lg d-flex flex-column align">
          <draggable
            class="ajust overflow-y-auto"
            :name="'flip-list'"
            v-model="lists[index].items"
            v-bind="dragOptions"
            :move="onMove"
          >
            <transition-group class="pa-4" tag="ul">
              <v-card
                class="list-group-item pa-3 mt-3 rounded-lg d-flex justify-space-between align"
                v-for="(element, index) in list.items"
                :key="index"
              >
                <div class="over" v-if="list.id == 2"></div>
                <v-icon color="green" v-if="list.id == 2" style="z-index: 20">
                  mdi-check
                </v-icon>
                <v-toolbar-title> {{ element.name }}</v-toolbar-title>

                <div>
                  <v-icon
                    @click.stop="
                      dialog = true;
                      editInput = element.name;
                      selectedItem = element;
                    "
                  >
                    mdi-pencil-outline
                  </v-icon>
                  <v-icon
                    @click.stop="
                      dialogDelete = true;
                      deleteProps.item = element;
                      deleteProps.list = list.items;
                    "
                    color="red"
                  >
                    mdi-trash-can-outline
                  </v-icon>
                </div>
              </v-card>
            </transition-group>
          </draggable>

          <v-text-field
            class="text mx-4"
            tag="div"
            v-if="list.id != 2"
            v-model="list.newTask"
            label="What are you working on?"
            solo
            @keydown.enter="addItem(list)"
          >
            <template v-slot:append>
              <v-fade-transition>
                <v-icon
                  v-if="list.newTask"
                  @click="addItem(list)"
                  color="green"
                >
                  mdi-plus-circle
                </v-icon>
              </v-fade-transition>
            </template>
          </v-text-field>
        </v-card>
      </v-col>
    </div>

    <v-dialog
      transition="dialog-bottom-transition"
      max-width="600"
      v-model="dialog"
    >
      <template>
        <v-card>
          <v-toolbar color="primary" dark> <h2>Edit</h2></v-toolbar>
          <v-text-field
            class="text ma-4"
            tag="input"
            v-model="editInput"
            label="..."
            solo
            @keypress.enter="closeDialog()"
          >
            <template v-slot:append>
              <v-btn icon>
                <v-fade-transition>
                  <v-icon
                    v-if="editInput.length > 0 && editInput.length < 20"
                    @click="closeDialog()"
                    color="green"
                  >
                    mdi-check
                  </v-icon>
                  <v-icon
                    v-if="editInput.length <= 0 || editInput.length > 20"
                    color="red"
                  >
                    mdi-close
                  </v-icon>
                </v-fade-transition>
              </v-btn>
            </template>
          </v-text-field>
          <v-card-actions class="justify-end">
            <v-btn text @click="dialog = false">Close</v-btn>
          </v-card-actions>
        </v-card>
      </template>
    </v-dialog>

    <v-dialog
      transition="dialog-bottom-transition"
      max-width="600"
      v-model="dialogDelete"
    >
      <template>
        <v-card>
          <v-toolbar color="primary" dark>Delete Card</v-toolbar>
          <v-card-text>
            <div class="text-h4 pa-6">
              Delete the card "{{ deleteProps.item.name }}" ?
            </div>
          </v-card-text>
          <v-card-actions class="justify-space-between">
            <v-btn color="red" text @click="removeItem()">Delete</v-btn>
            <v-btn text @click="dialogDelete = false">Close</v-btn>
          </v-card-actions>
        </v-card>
      </template>
    </v-dialog>
  </v-container>
</template>

<script>
import draggable from "vuedraggable";

const message = [
  "vue.draggable",
  "draggable",
  "component",
  "for",
  "vue.js 2.0",
  "based",
  "on",
  "Sortablejs",
  "Sortablejs",
  "Sortablejs",
  "Sortablejs",
];
export default {
  name: "Home",
  components: {
    draggable,
  },
  data() {
    return {
      lists: [
        {
          title: "To do",
          id: 0,
          newTask: null,
          items: message.map((name) => {
            return { name, done: false };
          }),
        },
        { title: "Doing", id: 1, newTask: null, items: [] },
        { title: "Done", id: 2, newTask: null, items: [] },
      ],
      editable: true,
      isDragging: false,
      delayedDragging: false,
      selectedItem: {},
      editInput: null,
      controlDialog: true,
      dialog: false,
      dialogDelete: false,
      deleteProps: {
        item: {},
        list: [],
      },
    };
  },
  methods: {
    closeDialog() {
      this.editItem();
      this.dialog = false;
    },
    editItem() {
      if (this.editInput.length > 0 && this.editInput.length < 30) {
        this.selectedItem.name = this.editInput;
      }
    },
    addItem(list) {
      list.items.push({
        name: list.newTask,
        done: false,
      });
      list.newTask = null;
    },
    removeItem() {
      const index = this.deleteProps.list.indexOf(this.deleteProps.item);
      if (index > -1) {
        this.deleteProps.list.splice(index, 1);
      }
      this.dialogDelete = false;
    },
    onMove({ relatedContext, draggedContext }) {
      const relatedElement = relatedContext.element;
      const draggedElement = draggedContext.element;

      return (
        (!relatedElement || !relatedElement.fixed) && !draggedElement.fixed
      );
    },
  },
  computed: {
    dragOptions() {
      return {
        animation: 0,
        group: "description",
        disabled: !this.editable,
        ghostClass: "ghost",
      };
    },
  },
};
</script>

<style>
.over {
  position: absolute;
  width: 100%;
  height: 100%;
  background: rgba(255, 255, 255, 0.404);
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  border-radius: inherit;
  z-index: 10;
}

.ajust {
  height: 87% !important;
}

.ghost {
  opacity: 0.5;
  background: #c8ebfb !important ;
}

.list--area {
  height: calc(100vh - 100px);
  flex-grow: 0;
  align-items: center;
}

.text {
  align-items: flex-end !important;
}

.list {
  width: 100%;
  height: 95%;
  background: #f5f5f5;
}

ul {
  height: 100%;
  list-style: none;
}

.list-group-item {
  cursor: move;
}
</style>