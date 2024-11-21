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
    deletePost(id: string) {
      const db = ref(this.storage).value;

      if (db) {
        let mydoc = this.fakeData();
       //faire une boucle qui récupère tout les id afin de les supprimer à la suite
       //db.collection("posts").doc(mydoc.id).delete()

        db.get(id)
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

    //code donné par le prof
    async deleteData(id: string, rev: string) {
        console.log('Call deleteData', rev)
        const db = ref(this.storage).value
        if (!db) {
            console.log('Database not valid')
            return
        }
        try {
            await db.remove(id, rev)
            console.log('deleteData success')
            this.fetchData()
        } catch (error) {
            console.log('deleteData error', error)
        }
    },

    updateLocalDatabase() {
        const db = ref(this.storage).value
        if (db) {
            db.replicate.from
                .bind(this)(db)
                .on('complete', () => {
                    console.log('on replicate complete')
                    this.fetchData()
                })
                .on('error', function (error) {
                    console.log('error', error)
                })
        }
    },
 
    updateDistantDatabase() {
        // TODO
 
        const db = ref(this.storage).value;
    const remoteDB = 'http://admin:CouchDB@127.0.0.1:5984/post_insta';
 
    if (db) {
        db.replicate.to(remoteDB)
            .on('complete', () => {
                console.log('Replication to remoteDB complete');
            })
            .on('error', (error) => {
                console.error('Error replicating to remoteDB:', error);
            });
    } else {
        console.warn('Local database is not initialized.');
    }
    },
 
    watchRemoteDatabase() {
        // TODO
 
        const db = ref(this.storage).value;
        const remoteDB = 'http://admin:CouchDB@127.0.0.1:5984/post_insta';
 
    if (db) {
        const remoteChanges = new PouchDB(remoteDB).changes({
            since: 'now',
            live: true,
            include_docs: true
        });
 
        remoteChanges
            .on('change', (change) => {
                console.log('Change detected in remoteDB:', change);
                this.fetchData(); // Met à jour les données locales après une modification
            })
            .on('error', (err) => {
                console.error('Error watching remote database:', err);
            });
    } else {
        console.warn('Local database is not initialized.');
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
  <button @click="syncWithRemote()">Sync with remote</button>
  <ul>
    <li v-for="post in postsData" :key="post._id">
      <div class="ucfirst">
        {{ post.doc.post_name
        }}
        {{ post.doc.post_content
        }}<em
          style="font-size: x-small"
          v-if="post.doc.attributes?.creation_date"
        >
          - {{ post.doc.attributes?.creation_date }}
        </em>
        <button @click="deletePost(post._id)" > delete Post</button>
      </div>
    </li>
  </ul>
</template>
