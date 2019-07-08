<template>
  <div>
    <li
      v-for="file in files"
      :key="file.name"
      class="test"
      @click="getFileMeta(file)"
    >
      {{ file.name }}
      {{ file.tempo }}
    </li>
  </div>
</template>

<script>
const fs = window.require("fs");
const path = window.require("path");
const parser = require("../../parser");
import low from "lowdb";
import LocalStorage from "lowdb/adapters/LocalStorage";
const shortid = require("shortid");
const adapter = new LocalStorage("db");
const db = low(adapter);
db._.mixin({
  pushUnique: function(array, key, newEl) {
    if (array.findIndex(el => el[key] === newEl[key]) === -1) {
      array.push(newEl);
    }
    return array;
  }
});

const EXTENSION = ".flp";
const PROJET_FOLDER =
  "";
export default {
  data() {
    return {
      files: []
    };
  },
  mounted() {
    db.defaults({ projects: [] }).write();
    this.files = db.get("projects").value();

    fs.readdir(PROJET_FOLDER, "utf-8", (err, data) => {
      data
        .filter(file => path.extname(file).toLowerCase() === EXTENSION)
        .forEach(name => {
          db.get("projects")
            .pushUnique("name", {
              id: shortid.generate(),
              name
            })
            .write();
        });

      this.files = db.get("projects").value();
    });
  },
  methods: {
    getFileMeta: function(file) {
      parser.parseFile(
        path.join(PROJET_FOLDER, file.name),
        (err, projectInfo) => {
          if (err) throw err;
          delete projectInfo.channels;
          delete projectInfo.effectChannels;

          db.get("projects")
            .find({ name: file.name })
            .assign(projectInfo)
            .write();
        }
      );

      // How ?
      this.files = Object.assign({}, this.files, file);
    }
  }
};
</script>

<style scoped>
.title {
  color: #888;
  font-size: 18px;
  font-weight: initial;
  letter-spacing: 0.25px;
  margin-top: 10px;
}

.items {
  margin-top: 8px;
}

.item {
  display: flex;
  margin-bottom: 6px;
}

.item .name {
  color: #6a6a6a;
  margin-right: 6px;
}

.item .value {
  color: #35495e;
  font-weight: bold;
}
</style>
