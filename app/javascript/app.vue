<template>
  <draggable v-model="lists" :options="{group: 'lists'}" :animation="200" class="row dragArea" v-on:end="listMoved">
    <div v-for="(list, index) in lists" :key="list.id" class="col-3">
      <div class="list_content">
        <h6>{{ list.name }}</h6>
        <hr />
        <draggable v-model="list.cards" :options="{group: 'cards'}" :animation="200" class="dragArea" v-on:change="cardMoved">
          <div v-for="(card, index) in list.cards" :key="card.id" class="card card-body mb-3">
            {{ card.name }}
          </div>
        </draggable>

        <div class="inputs">
          <textarea v-model="messages[list.id]" class="form-control" ></textarea>
          <button  v-on:click="submitMessages(list.id)" class="btn btn-primary">Add</button>
        </div>
      </div>
    </div>
  </draggable>
</template>


<script>
import draggable from 'vuedraggable'

export default {
  components: { draggable },
  props: ["original_lists"],
  data: function() {
    return {
      messages: {},
      lists: this.original_lists,
    }
  },
  methods: {
    cardMoved: function(event) {
      const evt = event.added || event.moved
      if (evt == undefined) { return }

      const element = evt.element

      const list_index = this.lists.findIndex((list) => {
        return list.cards.find((card) => {
          return card.id === element.id
        })
      })

      var data = new FormData
      data.append("card[list_id]", this.lists[list_index].id)
      data.append("card[position]", evt.newIndex + 1)

      Rails.ajax({
        url: `/cards/${element.id}/move`,
        type: "PATCH",
        data: data,
        dataType: "json"
      })

      // const array1 = this.lists[list_index].cards;
      // array1.forEach(card => alert(`現在${this.lists[list_index].name}の段階のタスクは”${card.name}”`));
      // alert("頑張りましょう！！");
    },
    
    listMoved: function(event) {
      var data = new FormData
      data.append("list[position]", event.newIndex + 1)

      Rails.ajax({
        url: `/lists/${this.lists[event.newIndex].id}/move`,
        type: "PATCH",
        data: data,
        dataType: "json",
      })
    },
    submitMessages: function(list_id) {
      var data = new FormData
      data.append("card[list_id]", list_id)
      data.append("card[name]",  this.messages[list_id])

      Rails.ajax({
        url: "/cards",
        type: "POST",
        data: data,
        dataType: "json",
        success: (data) => {
          const index = this.lists.findIndex(item => item.id == list_id)
          this.lists[index].cards.push(data)
          this.messages[list_id] = undefined
        }
      })
    }
  }
}
</script>

<style scoped>
p {
  font-size: 2em;
  text-align: center;
}

.dragArea {
  min-height: 10px;
}
</style>
