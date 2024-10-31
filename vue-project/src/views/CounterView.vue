<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>

<script lang="ts">
import { ref } from "vue";
import PouchDB from "pouchdb";

declare interface Post {
  _id: string;
  doc: {
    post_name: string;
    post_content: string;
    attributes: {
      creation_date: string;
      author: string;
    };
  };
}

export default {
  data() {
    return {
      total: 0,
      postsData: [] as Post[],
      document: null as Post | null,
      storage: null as PouchDB.Database | null,
    };
  },

  mounted() {
    this.iniDatabase();
    this.fetchData();
  },

  methods: {
    inc() {
      this.total++;
    },

    iniDatabase() {
      //lancer une intisalisation de la base de données
      const db = new PouchDB("http://admin:CouchDB@127.0.0.1:5984/post_insta");

      if (db) {
        console.log("Connected to collection 'post_insta'");
      } else {
        console.warn("Something went wrong");
      }
      this.storage = db;
    },

    //permet de mettre un document (comme un table) dans la database
    putDocument(document: Post) {
      const db = ref(this.storage).value;
      if (db) {
        db.put(document)
          .then(() => {
            console.log("Add ok");
          })
          .catch((error) => {
            console.log("Add ko", error);
          });
      }
    },

    //Créer un document en dur. methode qui retourne toujours le même object
    fakeData() {
      return {
        //pas inclure _rev dans le document, car post génère un nouveau document et CouchDB n'acceptera pas un document avec un _rev déjà défini.
        post_name: "Matin",
        post_content: "Bonjour Martin Matin",
        attributes: {
          creation_date: "yesterday",
          author: "Agathe",
        },
        comments: [
          {
            comment: "Comment 1",
            author: "Author 1",
          },
          {
            comment: "Comment 2",
            author: "Author 2",
          },
          {
            comment: "Comment 3",
            author: "Author 3",
          },
          {
            comment: "Comment 3",
            author: "Author 3",
          },
        ],
      };
    },
    // methode qui permet de mettre mon doc dans la database
    putFakeData() {
      // ou en une ligne avec l'opérateur ternaire !Mais ne gère pas les erreurs
      //this.storage?.post(this.fakeData());
      const db = ref(this.storage).value;

      if (db) {
        db.post(this.fakeData())
          .then(() => {
            console.log("Add ok");
          })
          .catch((error) => {
            console.log("Add ko", error);
          });
      }
    },
    // methode qui permet de supprimer mon doc de la database
    deleteFakeData() {
      const db = ref(this.storage).value;

      if (db) {
        let mydoc = this.fakeData();
       //faire une boucle qui récupère tout les id afin de les supprimer à la suite
       //db.collection("posts").doc(mydoc.id).delete()
       
        db.get('79051f634e82bd7ad13d029697001232')
          .then(function (mydoc) {
            return db.remove(mydoc);
          })
          .then(() => {
            console.log("Remove ok");
          })
          .catch((error) => {
            console.log("Remove ko", error);
          });
      }
    },

    fetchData() {
      // récupérer tout les documents de ma database
      const storage = ref(this.storage);
      const self = this;
      if (storage.value) {
        storage.value
          .allDocs({
            include_docs: true,
            attachments: true,
          })
          .then(
            function (result: any) {
              console.log("fetchData success", result);
              self.postsData = result.rows;
            }.bind(this)
          )
          .catch(function (error: any) {
            console.log("fetchData error", error);
          });
      }
    },
  },
};
</script>

<template>
  <h1>Nombre de post: {{ postsData.length }}</h1>
  <button @click="putFakeData()">Add post</button>
  <button @click="deleteFakeData()">Delete post</button>
  <ul>
    <li v-for="post in postsData" :key="post._id">
      <div class="ucfirst">
        {{ post.doc.post_name
        }}<em
          style="font-size: x-small"
          v-if="post.doc.attributes?.creation_date"
        >
          - {{ post.doc.attributes?.creation_date }}
        </em>
      </div>
    </li>
  </ul>
</template>
