<template>
  <v-container>
    <div class="d-flex flex-column flex-md-row justify-space-between">
      <v-col
        cols="12"
        md="4"
        lg="3"
        v-for="(list, index) in lists"
        :key="index"
        class="align-center d-flex flex-column"
        style="height: calc(100vh - 100px)"
      >
        <h2>{{ list.title }}</h2>
        <v-card class="list rounded-lg d-flex flex-column align">
          <draggable
            style="height: 87%"
            class="overflow-y-auto"
            :name="'flip-list'"
            v-model="lists[index].items"
            v-bind="dragOptions"
            :move="onMove"
          >
            <transition-group class="pa-4" tag="ul">
              <v-card
                class="pa-3 mt-3 rounded-lg align-center"
                style="cursor: move"
                v-for="(element, indexx) in list.items"
                :key="indexx + 1"
              >
                <v-row class="align-center">
                  <div class="over" v-if="list.id == 2"></div>
                  <v-col>
                    <v-avatar class="rounded-circle">
                      <v-icon
                        color="green"
                        v-if="list.id == 2"
                        style="z-index: 20"
                      >
                        mdi-check
                      </v-icon>
                      <img
                        @click.stop="
                          dialogAvatar = true;
                          avatarURL = element.avatar;
                          selectedItem = element;
                        "
                        style="cursor: pointer"
                        v-if="list.id != 2"
                        :src="element.avatar"
                        alt="Avatar"
                      />
                    </v-avatar>
                  </v-col>
                  <v-col
                    order-sm="3"
                    order-md="2"
                    order-xl="3"
                    class="d-flex justify-end"
                  >
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
                  </v-col>

                  <v-col
                    order-sm="2"
                    order-md="3"
                    order-xl="2"
                    md="12"
                    xl="6"
                    class="text-md-center"
                  >
                    {{ element.name }}</v-col
                  >
                </v-row>
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
            class="ma-4"
            tag="input"
            v-model="editInput"
            label="Name"
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

    <v-dialog
      transition="dialog-bottom-transition"
      max-width="600"
      v-model="dialogAvatar"
    >
      <template>
        <v-card>
          <v-toolbar color="primary" dark> <h2>Edit Avatar</h2></v-toolbar>
          <v-text-field
            class="ma-4"
            tag="input"
            v-model="avatarURL"
            label="URL"
          >
            <template v-slot:append>
              <v-fade-transition>
                <v-icon v-if="avatarURL.length > 0" color="green">
                  mdi-check
                </v-icon>
              </v-fade-transition>
            </template>
          </v-text-field>
          <v-col cols="12 justify-center d-flex">
            <img class="text-center" :src="avatarURL" alt="" width="30%" />
          </v-col>
          <v-card-actions class="justify-space-between">
            <v-btn
              color="success"
              text
              @click="
                selectedItem.avatar = avatarURL;
                dialogAvatar = false;
              "
              >Save</v-btn
            >
            <v-btn text @click="dialogAvatar = false">Close</v-btn>
          </v-card-actions>
        </v-card>
      </template>
    </v-dialog>
  </v-container>
</template>

<script>
import draggable from "vuedraggable";

const message = [
  {
    name: "vue.draggable",
    avatar:
      "https://avataaars.io/?avatarStyle=Circle&topType=LongHairBigHair&accessoriesType=Prescription02&hairColor=Platinum&facialHairType=MoustacheMagnum&facialHairColor=BrownDark&clotheType=Hoodie&clotheColor=Gray01&eyeType=Squint&eyebrowType=Default&mouthType=ScreamOpen&skinColor=DarkBrown",
  },
  {
    name: "draggable",
    avatar:
      "https://avataaars.io/?avatarStyle=Circle&topType=ShortHairTheCaesar&accessoriesType=Prescription01&hairColor=BlondeGolden&facialHairType=MoustacheMagnum&facialHairColor=Red&clotheType=ShirtVNeck&clotheColor=Black&eyeType=Close&eyebrowType=Default&mouthType=Twinkle&skinColor=Pale",
  },
  {
    name: "component",
    avatar:
      "https://avataaars.io/?avatarStyle=Circle&topType=ShortHairTheCaesar&accessoriesType=Sunglasses&hairColor=Auburn&facialHairType=BeardMedium&facialHairColor=Black&clotheType=BlazerShirt&clotheColor=PastelRed&eyeType=Surprised&eyebrowType=UnibrowNatural&mouthType=Twinkle&skinColor=Tanned",
  },
  {
    name: "for",
    avatar:
      "https://avataaars.io/?avatarStyle=Circle&topType=LongHairDreads&accessoriesType=Kurt&hairColor=Black&facialHairType=BeardLight&facialHairColor=Auburn&clotheType=ShirtCrewNeck&clotheColor=Pink&eyeType=Hearts&eyebrowType=Angry&mouthType=Sad&skinColor=Brown",
  },
  {
    name: "vue.js 2.0",
    avatar:
      "https://avataaars.io/?avatarStyle=Circle&topType=LongHairStraightStrand&accessoriesType=Kurt&hairColor=Auburn&facialHairType=Blank&clotheType=ShirtVNeck&clotheColor=Pink&eyeType=Wink&eyebrowType=UpDown&mouthType=Concerned&skinColor=Black",
  },
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
          items: message.map((element) => {
            return { name: element.name, avatar: element.avatar };
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
      avatarURL: null,
      dialog: false,
      dialogAvatar: false,
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
      if (list.newTask.length > 0) {
        list.items.push({
          name: list.newTask,
          avatar:
            "https://avataaars.io/?avatarStyle=Circle&topType=LongHairStraightStrand&accessoriesType=Kurt&hairColor=Auburn&facialHairType=Blank&clotheType=ShirtVNeck&clotheColor=Pink&eyeType=Wink&eyebrowType=UpDown&mouthType=Concerned&skinColor=Black",
        });
      }
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
.ghost {
  opacity: 0.5;
  background: #c8ebfb !important ;
}
.list {
  width: 100%;
  height: 95%;
}

ul {
  height: 100%;
  list-style: none;
}
</style>