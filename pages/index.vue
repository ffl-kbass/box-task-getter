<template>
  <div class="w-full h-full flex items-cetner justify-center flex-col p-5">
    <table class="rounded shadow-md ring-2 ring-gray-200 rounded overflow-hidden">
      <tr class="text-white bg-blue-600">
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
        );
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
        );

      console.log("Fetching Folder");
      this.entries = this.entries.concat(folder.entries);
      if (this.count) this.count = folder.total_count;
      this.curmarker = this.nextmarker;
      if (folder.next_marker) this.nextmarker = folder.next_marker;
    },
    async fetchFiles(folder) {
      console.log("Fetch Files");
      for (let i = 0; i < folder.length; i++) {
        console.log(i);
        if (folder[i].type == "folder") continue;
        console.log("Fetching File");
        var task = await this.$axios
          .$get("https://api.box.com/2.0/files/" + folder[i].id + "/tasks/", {
            headers: {
              "Access-Control-Allow-Origin": "*",
              Authorization: "Bearer " + process.env.API_KEY,
            },
          })
          .catch((error) => {
            console.log(error)
          })
          .then((res) => {
            console.log(res);
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
    },
  },
};
</script>
<style>
tr.row:nth-child(odd){
  background-color: #bbb;
}
</style>