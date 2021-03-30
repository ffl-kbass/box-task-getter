<template>
  <div class="w-full h-full flex items-cetner justify-center flex-col p-5">
    <div v-if='status.length' class="w-full flex items-center justify-center mb-5">
        <h1 class="text-xs text-center text-gray-700 bg-gray-200 rounded-full py-1 px-2 mb-5 lg:text-sm lg:py-2 lg:px-4 lg:mb-0">{{status}}</h1>
      </div>
    <table class="rounded shadow-md ring-2 ring-indigo-200 rounded overflow-hidden">
      <tr class="text-white bg-indigo-500">
        <th class="p-2">ID</th>
        <th class="p-2">File</th>
        <th class="p-2">Created By</th>
        <th class="p-2">Created At</th>
        <th class="p-2">Message</th>
        <th class="p-2">Assigned To</th>
        <th class="p-2">Completed</th>
      </tr>
      <tr
        v-for="{
          task_id,
          name,
          created_by,
          created_at,
          message,
          completed,
          assigned,
          id,
        } in file"
        :key="id"
        class="p-2 row"
      >
        <td class="p-2">{{ task_id }}</td>
        <td class="p-2">{{ name }}</td>
        <td class="p-2">{{ created_by.name }}</td>
        <td class="p-2">{{ created_at }}</td>
        <td class="p-2">{{ message }}</td>
        <td class="p-2">
          <span v-for="(item, id) of assigned" :key="id">{{ item }}<span v-if="id % 2 === 0 && assigned.length != 1">,</span></span>
        </td>
        <td class="p-2">{{ completed }}</td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      curmarker: "",
      nextmarker: "",
      entries: [],
      file: [],
      status: '',
    };
  },
  created() {
    this.start();
  },
  methods: {
    async start() {
      await this.fetchFolder();
      
      while (this.curmarker != this.nextmarker) {
        await this.fetchFolder();
      }
      await this.fetchFiles(this.entries);
    },
    async fetchFolder() {
      this.status="Fetching Folder"
      if (this.marker == "")
        var folder = await this.$axios.$get(
          "https://api.box.com/2.0/folders/" +
            process.env.FOLDER_ID +
            "/items?usemarker=true",
          {
            headers: {
              "Access-Control-Allow-Origin": "*",
              Authorization: "Bearer " + process.env.API_KEY,
            },
          }
        ).catch((error) => {
            console.log(error)
            this.status="There seems to have been an error. Please check if your folder id and API key are correct."
            return;
        });
      else
        var folder = await this.$axios.$get(
          "https://api.box.com/2.0/folders/" +
            process.env.FOLDER_ID +
            "/items?usemarker=true&marker=" +
            this.nextmarker,
          {
            headers: {
              "Access-Control-Allow-Origin": "*",
              Authorization: "Bearer " + process.env.API_KEY,
            },
          }
        ).catch((error) => {
            console.log(error)
            this.status="There seems to have been an error. Please check if your folder id and API key are correct."
            return;
        });

      this.entries = this.entries.concat(folder.entries);
      if (this.count) this.count = folder.total_count;
      this.curmarker = this.nextmarker;
      if (folder.next_marker) this.nextmarker = folder.next_marker;
    },
    async fetchFiles(folder) {
      this.status="Fetching Tasks"
      for (let i = 0; i < folder.length; i++) {
        if (folder[i].type == "folder") continue;
        var task = await this.$axios
          .$get("https://api.box.com/2.0/files/" + folder[i].id + "/tasks/", {
            headers: {
              "Access-Control-Allow-Origin": "*",
              Authorization: "Bearer " + process.env.API_KEY,
            },
          })
          .catch((error) => {
            console.log(error)
            this.status="There seems to have been an error. Please check if your folder id and API key are correct."
            return;
          })
          .then((res) => {
            let assigned_arr = [];
            let task_arr = [];
            if (res.entries.length > 0) {
              var obj = {};
              for (let k = 0; k < res.entries.length; k++) {
                obj = {};
                obj.name = folder[i].name;
                obj.task_id = res.entries[k].id;
                obj.created_by = res.entries[k].created_by;
                obj.created_at = res.entries[k].created_at;
                obj.message = res.entries[k].message;
                obj.completed = res.entries[k].is_completed;
                for (
                  let j = 0;
                  j < res.entries[k].task_assignment_collection.entries.length;
                  j++
                ) {
                  assigned_arr.push(
                    res.entries[k].task_assignment_collection.entries[j]
                      .assigned_to.name
                  );
                }
                obj.assigned = assigned_arr;
                assigned_arr = [];
                this.file.push(obj);
              }
            } else {
              // var obj = {};
              // obj.name = folder[i].name
              // this.file.push(obj)
            }
          });
      }
      this.status=""
    },
  },
};
</script>
<style>
tr.row:nth-child(odd){
  background-color: #f1f1f1;
}
</style>