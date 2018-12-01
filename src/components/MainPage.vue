<template>
  <div class="hello">
    <div class="start" v-if="!docTransparencia">
      <h1>{{ hello }}</h1>
      <h2>{{ msg }}</h2>
      <img src="../assets/logo.gif">
    </div>
    <label for="codigo">Código da despesa</label>
    <input id="codigo" class="group" type="text">
    <p></p>
    <button :disabled="loading" v-on:click="getSpecificDocument(codigo), loading=true">Procurar despesa pública</button>
    <p></p>
    <table align="center" v-if="docTransparencia">
      <thead>
        <tr v-bind:key="topic, value" v-for="(value, topic) in (Object.keys(docTransparencia), docTransparencia)">
          <div v-if="topic != 'codigoOrgao' && topic != 'codigoOrgaoSuperior'">
            <th>{{removeCamelCase(topic)}}</th>
            <td>
              <div v-if="topic == 'valor'">R$ {{trim(value)}}</div>
              <div v-else>{{trim(value)}}</div>
            </td>
          </div>
        </tr>
      </thead>
    </table>
    <div class="row">
      <div class="column" v-bind:key="comment" v-for="comment in comments">
        <div class="card">
          <div v-if="edit_id != comment._id">
            <h3>{{comment.author}}</h3>
            <p>{{comment.comment}}</p>
            <button id="delete" class="round-button" v-on:click="deleteComment(comment._id)"><img id="menu" src="../assets/delete.png"></button>
            <button id="edit" class="round-button" v-on:click="edit_id = comment._id"><img id="menu" src="../assets/edit.png"></button>
          </div>
          <div v-if="edit_id == comment._id">
            <input id="author" type="text" :placeholder="comment.author" v-on:input="edit_auth = $event.target.value">
            <input id="comment" type="text" :placeholder="comment.comment" v-on:input="edit_com = $event.target.value">
            <button id="save" class="submit" v-on:click="editComment(comment._id, edit_auth, edit_com, comment)">Salvar</button>
            <button id="cancel" class="submit" v-on:click="edit_id = ''">Cancelar</button>
          </div>
        </div>
      </div>
      <div class="column" v-if="docTransparencia">
        <div class="card">
          <input id="author" type="text" placeholder="Autor" v-on:input="auth = $event.target.value">
          <input id="comment" type="text" placeholder="Comentário" v-on:input="com = $event.target.value">
          <button id="create" class="submit" v-on:click="postComment(auth, com)">Comentar</button>
        </div>
      </div>
    </div>
    <router-view/>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'MainPage',
  data () {
    return {
      hello: 'Bem-vindo ao Gossiper',
      msg: 'O site para comentar nas despesas públicas do portal da transparência',
      codigo: '',
      docTransparencia: '',
      comments: '',
      auth: '',
      com: '',
      edit_auth: '',
      edit_com: '',
      loading: false,
      edit_id: ''
    }
  },

  methods: {
    getSpecificDocument: function () {
      this.codigo = document.getElementById('codigo').value.toString()
      // eslint-disable-next-line
      axios.get('http://localhost:3000/' + this.codigo + '/comments').then((response) => {
        this.docTransparencia = ''
        this.comments = ''
        this.docTransparencia = response.data
        this.comments = this.docTransparencia.comments
        delete this.docTransparencia.comments
        this.loading = false
      }).catch((reason) => {
        this.docTransparencia = ''
        this.comments = ''
        this.loading = false
      })
    },

    postComment: function (author, comment) {
      axios.post('http://localhost:3000/' + this.codigo + '/comments', {author: author, comment: comment}).then((response) => {
        this.comments.push(response.data)
        this.loading = false

        document.getElementById('author').value = ''
        document.getElementById('comment').value = ''
        this.auth = ''
        this.com = ''
      }).catch((reason) => {
        this.loading = false
      })
    },

    editComment: function (commentId, author, comment, prevComment) {
      console.log({author: author})
      if (comment == null || comment === '') comment = prevComment.comment
      if (author == null || author === '') author = prevComment.author

      axios.put('http://localhost:3000/' + this.codigo + '/comments/' + commentId, {author: author, comment: comment}).then((response) => {
        const editIndex = this.comments.findIndex((element, index, array) => {
          return element._id === prevComment._id
        })

        this.comments[editIndex].author = response.data.author
        this.comments[editIndex].comment = response.data.comment

        this.edit_id = ''
        this.edit_auth = ''
        this.edit_com = ''
      })
    },

    deleteComment: function (commentId) {
      axios.delete('http://localhost:3000/' + this.codigo + '/comments/' + commentId).then((response) => {
        var aux = this.comments
        const deleteIndex = aux.findIndex((element, index, array) => {
          return element._id === response.data._id
        })

        aux.splice(deleteIndex, 1)
        this.comments = aux
      })
    },

    removeCamelCase: function (text) {
      var result = text.replace(/([A-Z])/g, ' $1')
      return result.charAt(0).toUpperCase() + result.slice(1)
    },

    trim: function (text) {
      if (text != null) return text.toString().trim()
      else return null
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1{
  margin-top: 60px;
}
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
#menu{
  width: 20px;
  border-radius: 20%;
  float:left;
}
button {
  vertical-align: 20px;
  padding: 15px 25px;
  font-size: 24px;
  text-align: center;
  cursor: pointer;
  outline: none;
  color: #fff;
  background-color: #42b983;
  border: none;
  border-radius: 15px;
  box-shadow: 0 9px #389b6e;
}
button:hover {
  background-color: #41b681;
}
button:active {
  background-color: #41b681;
  box-shadow: 0 5px #389b6e;
  transform: translateY(4px);
}
button:disabled {
  background-color: #699;
  box-shadow: 0 5px #999;
  transform: translateY(4px);
  cursor: default;
}
table {
  border: 5px solid #42b983;
  border-radius: 3px;
  background-color: #fff;
  font-size: 16px;
  display: inline-block;
}
th {
  background-color: #42b983;
  color: rgba(255,255,255,0.66);
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
td {
  background-color: #f9f9f9;
}
th, td {
  min-width: 170px;
  padding: 10px 15px;
  text-align: left;
}
th, td, button, h1, h2{
  font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
}
input{
  border-radius: 4px;
  border: 1px solid #ccc;
  width: 60%;
  max-width: 270px;
  margin: 0px auto;
  padding: 10px;
  position: relative;
  overflow: hidden;
  font-size: 20px;
}
input#codigo{
  margin: 10px auto;
  width: 30%;
}
input:focus{
  border-color: #7185ec;
}
input, label{
  display: block;
}
#create, #save, #cancel{
  width: 30%;
  padding: 6px;
}
#author, #comment{
  text-align: center;
}
label{
  font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
  font-size: 20px;
  margin-top: 60px;
  margin-left: -140px;
}
.submit{
  margin-top: 5px;
  margin-bottom: 5px;
  font-size: 20px;
}
/* Float four columns side by side */
.column {
  display:inline-block;
  width: 25%;
  padding: 0 10px;
}

/* Remove extra left and right margins, due to padding */
.row {
  margin-top: 10px;
  width: 100%;
  text-align: center;
}

/* Clear floats after the columns */
.row:after {
  content: "";
  display: table;
  clear: both;
}

/* Responsive columns */
@media screen and (max-width: 600px) {
  .column {
    width: 100%;
    display: block;
    margin-bottom: 20px;
  }
}

/* Style the counter cards */
.card {
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
  padding: 16px;
  text-align: center;
  background-color: #f1f1f1;
  border: 5px solid #42b983;
  margin-bottom: 10px;
  overflow-wrap: break-word;
}

.round-button {
  float: right;
  padding: 16px;
  margin-top: -50px;
}
.round-button:active {
  transform: translateY(4px);
}
.round-button:disabled {
  transform: translateY(4px);
  cursor: default;
}
#delete{
  background-color: #b82727; /* Red */
  box-shadow: 0 7px #8a2222;
}
#delete:hover{
  background-color: #aa2f2f;
}
#delete:active{
  background-color: #aa2f2f;
  box-shadow: 0 5px #8a2222;
}
#delete:disabled{
  background-color: #955;
  box-shadow: 0 5px #999;
}

#edit{
  margin-right: 10px;
  background-color: #42b983; /* Red */
  box-shadow: 0 7px #389b6e;
}
#edit:hover{
  background-color: #41b681;
}
#edit:active{
  background-color: #41b681;
  box-shadow: 0 5px #389b6e;
}
#edit:disabled{
  background-color: #699;
  box-shadow: 0 5px #999;
}
</style>
