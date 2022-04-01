<template>
  <div class="post-wrapper" v-if="post">
    <!-- LOADER -->
    <div v-if="loading">
      <div class="loading-container">
        <Loader />
      </div>
    </div>
    <!-- ERROR MESSAGE -->
    <div
      class="error-container"
      v-if="!loading && errorMessage"
      :innerHTML="errorMessage"
    ></div>
    <div class="post-container">
      <header class="post-header">
        <h2>{{ post.title }}</h2>
      </header>
      <div class="text-muted updated">
        <div class="user">
          <img
            :src="post.user_image"
            class="user-profile"
            v-show="!loading && !errorMessage"
          />
          <span v-show="!loading && !errorMessage">{{ post.created_by }}</span>
        </div>
        <div class="date">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="20"
            height="20"
            fill="currentColor"
            class="bi bi-calendar-check text-muted"
            viewBox="0 0 16 16"
            v-show="!loading && !errorMessage"
          >
            <path
              d="M10.854 7.146a.5.5 0 0 1 0 .708l-3 3a.5.5 0 0 1-.708 0l-1.5-1.5a.5.5 0 1 1 .708-.708L7.5 9.793l2.646-2.647a.5.5 0 0 1 .708 0z"
            />
            <path
              d="M3.5 0a.5.5 0 0 1 .5.5V1h8V.5a.5.5 0 0 1 1 0V1h1a2 2 0 0 1 2 2v11a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2V3a2 2 0 0 1 2-2h1V.5a.5.5 0 0 1 .5-.5zM1 4v10a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1V4H1z"
            />
          </svg>
          <span v-show="!loading && !errorMessage">
            {{ moment(post.updatedAt).format("MMM DD, YYYY") }}</span
          >
        </div>
      </div>
      <div class="post-image" v-show="!loading && !errorMessage">
        <img :src="post.main_image" class="main-image" />
      </div>
      <article class="post-content">
        <h3>{{ post.subtitle }}</h3>
      </article>
      <div class="post-markup" v-html="post.desc"></div>
      <div class="action-buttons" v-if="currentUser">
        <button
          type="button"
          class="edit-btn"
          data-bs-toggle="modal"
          data-bs-target="#editModal"
          v-if="currentUser.username == post.created_by"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="16"
            height="16"
            fill="currentColor"
            class="bi bi-pencil-fill"
            viewBox="0 0 16 16"
          >
            <path
              d="M12.854.146a.5.5 0 0 0-.707 0L10.5 1.793 14.207 5.5l1.647-1.646a.5.5 0 0 0 0-.708l-3-3zm.646 6.061L9.793 2.5 3.293 9H3.5a.5.5 0 0 1 .5.5v.5h.5a.5.5 0 0 1 .5.5v.5h.5a.5.5 0 0 1 .5.5v.5h.5a.5.5 0 0 1 .5.5v.207l6.5-6.5zm-7.468 7.468A.5.5 0 0 1 6 13.5V13h-.5a.5.5 0 0 1-.5-.5V12h-.5a.5.5 0 0 1-.5-.5V11h-.5a.5.5 0 0 1-.5-.5V10h-.5a.499.499 0 0 1-.175-.032l-.179.178a.5.5 0 0 0-.11.168l-2 5a.5.5 0 0 0 .65.65l5-2a.5.5 0 0 0 .168-.11l.178-.178z"
            />
          </svg>
        </button>
        <button
          type="button"
          class="delete-btn"
          data-bs-toggle="modal"
          data-bs-target="#deleteModal"
          v-if="currentUser.username == post.created_by"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="16"
            height="16"
            fill="currentColor"
            class="bi bi-x-circle-fill"
            viewBox="0 0 16 16"
          >
            <path
              d="M16 8A8 8 0 1 1 0 8a8 8 0 0 1 16 0zM5.354 4.646a.5.5 0 1 0-.708.708L7.293 8l-2.647 2.646a.5.5 0 0 0 .708.708L8 8.707l2.646 2.647a.5.5 0 0 0 .708-.708L8.707 8l2.647-2.646a.5.5 0 0 0-.708-.708L8 7.293 5.354 4.646z"
            />
          </svg>
        </button>
      </div>
    </div>
    <div class="comment-container">
      <h5 v-if="currentUser" v-show="!loading && !errorMessage">Comments</h5>
      <form
        @submit.prevent="addComment(this.postId)"
        class="text-container"
        v-if="currentUser"
      >
        <textarea
          required
          type="text"
          class="form-control text shadow-sm"
          id="comments"
          placeholder="Write a comment..."
          v-model="newComment"
          v-show="!loading && !errorMessage"
        />
        <div class="comment-btn" v-if="currentUser">
          <button
            class="comment-post-btn"
            type="submit"
            v-show="!loading && !errorMessage"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              fill="currentColor"
              class="bi bi-send-fill"
              viewBox="0 0 16 16"
            >
              <path
                d="M15.964.686a.5.5 0 0 0-.65-.65L.767 5.855H.766l-.452.18a.5.5 0 0 0-.082.887l.41.26.001.002 4.995 3.178 3.178 4.995.002.002.26.41a.5.5 0 0 0 .886-.083l6-15Zm-1.833 1.89L6.637 10.07l-.215-.338a.5.5 0 0 0-.154-.154l-.338-.215 7.494-7.494 1.178-.471-.47 1.178Z"
              />
            </svg>
          </button>
        </div>
      </form>
      <div class="comments-header">
        <h5 v-show="!loading && !errorMessage">Recent Comments</h5>
      </div>
      <div class="comments-wrapper">
        <div
          class="comments"
          v-for="comment in post.comments"
          :key="comment._id"
        >
          <div class="comment-info">
            <div class="comment-content">
              <h6>{{ comment.posted_by }}</h6>
              <p class="comment-content text-muted">{{ comment.content }}</p>
            </div>
            <div class="comment-action" v-if="currentUser">
              <button
                class="comment-delete-btn"
                @click.prevent="deleteComment(this.postId, comment._id)"
                v-if="currentUser.username == comment.posted_by"
              >
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="16"
                  height="16"
                  fill="currentColor"
                  class="bi bi-x-circle-fill"
                  viewBox="0 0 16 16"
                >
                  <path
                    d="M16 8A8 8 0 1 1 0 8a8 8 0 0 1 16 0zM5.354 4.646a.5.5 0 1 0-.708.708L7.293 8l-2.647 2.646a.5.5 0 0 0 .708.708L8 8.707l2.646 2.647a.5.5 0 0 0 .708-.708L8.707 8l2.647-2.646a.5.5 0 0 0-.708-.708L8 7.293 5.354 4.646z"
                  />
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>
      <div v-if="post.comments == 0" class="comment-placeholder">
        <h6 class="text-muted">No comments have been posted yet.</h6>
      </div>
    </div>
  </div>
  <footer class="blog-footer">
    <div class="footer-copyright">
      <h6 class="copyright">Copyright © 2022 The Mental Mind</h6>
    </div>
  </footer>

  <!-- EDIT MODAL -->

  <div
    class="modal fade"
    id="editModal"
    data-bs-backdrop="static"
    data-bs-keyboard="false"
    tabindex="-1"
    aria-labelledby="staticBackdropLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="staticBackdropLabel">Update post.</h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          <form
            @submit.prevent="updatePost(this.postId)"
            class="row create-form"
          >
            <div class="col-md-12">
              <label for="main-image" class="form-label mt-1">Image *</label>
              <input
                required
                type="text"
                v-model="updatedPost.main_image"
                placeholder="Insert image link..."
                class="form-control"
                id="main-image"
                data-bs-toggle="tooltip"
                data-bs-placement="top"
                title="Images should be uploaded using image links. Example: https://www.google.com/url?sa=i&url=https%3A%2F%2Fpixabay.com... Recommended image size is 1200 x 800."
              />
            </div>
            <div class="col-md-12">
              <label for="title" class="form-label">Title *</label>
              <input
                required
                type="text"
                v-model="updatedPost.title"
                class="form-control"
                id="title"
              />
            </div>
            <div class="col-md-12">
              <label for="subtitle" class="form-label">Subtitle *</label>
              <input
                required
                type="text"
                v-model="updatedPost.subtitle"
                class="form-control"
                id="subtitle"
              />
            </div>
            <div class="col-md-12">
              <label for="desc" class="form-label">Content *</label>
              <textarea
                required
                type="text"
                v-model="updatedPost.desc"
                class="form-control"
                id="desc"
                placeholder="This text area can render HTML :)"
              />
              <div class="col-md-12 info-message">
                <h6>Note: All fields marked with * are required.</h6>
              </div>
            </div>
            <div class="modal-footer">
              <button type="button" class="cancel-btn" data-bs-dismiss="modal">
                Cancel
              </button>
              <button
                @click.prevent="addPost"
                class="update-btn"
                type="button"
                :disabled="loading"
              >
                <span v-show="!loading">Upload</span>
                <span
                  class="spinner-border spinner-border-sm"
                  role="status"
                  aria-hidden="true"
                  v-show="loading"
                ></span>
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>

  <!-- DELETE MODAL -->
  <div
    class="modal fade"
    id="deleteModal"
    data-bs-backdrop="static"
    data-bs-keyboard="false"
    tabindex="-1"
    aria-labelledby="staticBackdropLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="staticBackdropLabel">
            Are you sure you want to delete this post?
          </h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-footer">
          <button type="button" class="cancel-btn" data-bs-dismiss="modal">
            Cancel
          </button>
          <button
            type="button"
            class="post-delete-btn"
            @click.prevent="deletePost(this.postId)"
            :disabled="loading"
          >
            <span v-show="!loading">Proceed</span>
            <span
              class="spinner-border spinner-border-sm"
              role="status"
              aria-hidden="true"
              v-show="loading"
            ></span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import PostService from "../services/post.service";
import Loader from "../components/Loader.vue";
import moment from "moment";
const url = "https://final-blog-api.herokuapp.com/posts/";

export default {
  name: "Postview",
  components: {
    Loader,
  },
  data() {
    return {
      loading: false,
      postId: this.$route.params.id,
      post: {},
      updatedPost: {
        main_image: "",
        title: "",
        subtitle: "",
        desc: ``,
      },
      errorMessage: null,
      newComment: "",
      moment,
    };
  },
  computed: {
    currentUser() {
      return this.$store.state.auth.user;
    },
  },
  mounted() {
    this.moment = moment;
    this.loading = true;
    PostService.getOnePost(this.postId)
      .then((res) => {
        this.post = res.data;
        this.loading = false;
      })
      .catch((err) => {
        this.errorMessage = `
        <h1>Oops!</h1>
        <p>Seems like there was an error. Try refreshing this page or checking your internet connection.</p>
        `;
        this.loading = false;
      });
  },
  methods: {
    async deletePost(postId) {
      const headers = {
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${
            JSON.parse(localStorage.getItem("user")).access
          }`,
        },
      };
      const postedBy = this.post.created_by;
      const new_url = `${url}${postId}`;
      const username = JSON.parse(localStorage.getItem("user")).username;
      if (username !== postedBy) {
        return alert("You cannot delete this post!");
      }
      try {
        await axios.delete(new_url, headers, this.post).then(() => {
          this.$router.push("/");
        });
      } catch (error) {
        console.error(error);
      }
    },
    async updatePost(postId) {
      const postedBy = this.post.created_by;
      const username = JSON.parse(localStorage.getItem("user")).username;
      if (username !== postedBy) {
        return alert("You cannot update this post!");
      }
      try {
        this.loading = true;
        fetch(`${url}${postId}`, {
          method: "PUT",
          body: JSON.stringify({
            main_image: this.updatedPost.main_image,
            title: this.updatedPost.title,
            subtitle: this.updatedPost.subtitle,
            catergory: this.updatedPost.catergory,
            desc: this.updatedPost.desc,
          }),
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${
              JSON.parse(localStorage.getItem("user")).access
            }`,
          },
        })
          .then((res) => res.json())
          .then(() => {
            this.loading = false;
            location.reload();
          });
      } catch (error) {
        console.error(error);
      }
    },
    async addComment(postId) {
      try {
        this.loading = true;
        fetch(`${url}${postId}/comments/create`, {
          method: "POST",
          body: JSON.stringify({
            content: this.newComment,
          }),
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${
              JSON.parse(localStorage.getItem("user")).access
            }`,
          },
        })
          .then((res) => res.json())
          .then(() => {
            this.loading = false;
            location.reload();
          });
      } catch (error) {
        console.error(error);
      }
    },
    async deleteComment(postId, commentId) {
      try {
        this.loading = true;
        fetch(`${url}${postId}/comments/delete/${commentId}`, {
          method: "DELETE",
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${
              JSON.parse(localStorage.getItem("user")).access
            }`,
          },
        })
          .then((res) => res.json())
          .then(() => {
            this.loading = false;
            location.reload();
          });
      } catch (error) {
        console.error(error);
      }
    },
  },
};
</script>

<style scoped>
.post-wrapper {
  display: flex;
  flex-direction: column;
  row-gap: 1rem;
  width: 100%;
  margin: 5rem 0;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}
.post-header {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  flex-direction: column;
  width: 50%;
  margin-top: 1rem;
}
.post-header h2 {
  font-size: 60px;
  font-weight: 700;
  width: 100%;
}
/* TEXT MARKUP */
.post-content {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  flex-direction: column;
  width: 50%;
  row-gap: 1rem;
}
.post-content h3 {
  font-size: 40px;
}
.post-markup {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  flex-direction: column;
  width: 50%;
  row-gap: 0;
}
.post-content p {
  font-family: "Cabin", sans-serif;
}
.post-markup p {
  font-family: "Cabin", sans-serif;
}
.post-markup h1 {
  font-size: 36px;
  font-weight: 600;
  width: 100%;
}
.post-markup h2 {
  font-size: 34px;
  font-weight: 600;
  width: 100%;
}
.post-markup h3 {
  font-size: 32px;
  font-weight: 600;
  width: 100%;
}
.post-markup h4 {
  font-size: 30px;
  font-weight: 600;
  width: 100%;
}
.post-markup h5 {
  font-size: 28px;
  font-weight: 600;
  width: 100%;
}
.post-markup h6 {
  font-size: 26px;
  font-weight: 600;
  width: 100%;
}

.post-image {
  width: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.post-image img {
  width: 100%;
  object-fit: cover;
  max-height: 600px;
  object-position: center;
}
.post-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  row-gap: 1rem;
}
.action-buttons {
  display: flex;
  width: 50%;
  justify-content: flex-start;
  align-items: center;
  column-gap: 1rem;
}
.form-control {
  display: flex;
  flex-basis: 100%;
}
.text-container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  row-gap: 1rem;
  padding: 0;
  width: 50%;
}
textarea {
  min-height: 264px;
  padding: 10px;
}
.text {
  border: none;
}
.updated {
  display: flex;
  width: 50%;
  flex-direction: row;
  justify-content: flex-start;
  column-gap: 1rem;
  align-items: center;
  font-family: "Cabin", sans-serif;
  font-weight: 300;
  padding: 0;
  margin: 0;
}
.user,
.date {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: row;
  gap: 5px;
}
.comment-content p {
  font-family: "Cabin", sans-serif;
  font-size: 15px;
}
.loading-container {
  margin-top: 8rem;
}
.error-container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  text-align: center;
  margin-top: 8rem;
  width: 60%;
  font-family: "Cabin", sans-serif;
  font-weight: 600;
  margin: 0;
  padding: 0;
}
.comment-container {
  width: 50%;
  min-width: 20%;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  flex-direction: column;
  row-gap: 1rem;
  margin: 3rem 0;
  padding: 0;
}
#comments {
  min-height: 100px;
  width: 100%;
}
.create-form {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  row-gap: 0.5rem;
  padding: 0;
}
.modal-body {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  padding: 0;
}
.info-message {
  padding: 0;
  margin-top: 0.5rem;
}
.info-message h6 {
  color: #1f1f1f;
  padding: 0;
  margin: 0;
}
.comments-wrapper {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  flex-direction: column;
  row-gap: 1rem;
  padding: 0;
}
.comments {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-direction: column;
  row-gap: 0.5rem;
  width: 50%;
}
.comment-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-direction: row;
  width: 100%;
}
.comment-placeholder h6 {
  font-size: 16px;
  font-weight: 100;
}
.comment-btn {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  width: 100%;
  margin: 0;
  padding: 0;
}
.comment-delete-btn,
.comment-post-btn {
  min-width: 80px;
  padding: 5px;
  outline: none;
  border: none;
  background: none;
  color: #1f1f1f;
  transition: ease-in-out 500ms;
  display: flex;
  justify-content: flex-end;
  align-items: center;
}
.comment-delete-btn svg,
.comment-post-btn svg {
  color: #1f1f1f;
  width: 30px;
  height: 30px;
  transition: ease-in-out 500ms;
  filter: drop-shadow(0 0.125rem 0.25rem rgba(0, 0, 0, 0.075));
}
.comment-delete-btn svg:hover {
  transform: scale(1.1);
  color: red;
}
.comment-post-btn svg:hover {
  transform: scale(1.1);
  color: green;
}
.edit-btn,
.delete-btn {
  min-width: 60px;
  padding: 0;
  outline: none;
  border: none;
  background: transparent;
  color: #1f1f1f;
  transition: ease-in-out 500ms;
  display: flex;
  justify-content: flex-start;
  align-items: center;
}
.edit-btn svg,
.delete-btn svg {
  color: #1f1f1f;
  width: 30px;
  height: 30px;
  transition: ease-in-out 500ms;
  filter: drop-shadow(0 0.125rem 0.25rem rgba(0, 0, 0, 0.075));
}
.edit-btn svg:hover {
  color: green;
  transform: scale(1.1);
}
.delete-btn svg:hover {
  color: red;
  transform: scale(1.1);
}
.post-delete-btn:hover {
  color: green;
}
.cancel-btn,
.update-btn,
.post-delete-btn {
  min-width: 60px;
  padding: 0;
  margin: 0;
  outline: none;
  border: none;
  background: transparent;
  color: #1f1f1f;
  transition: ease-in-out 500ms;
  display: flex;
  justify-content: flex-end;
  align-items: center;
}
.cancel-btn:hover {
  color: red;
}
.update-btn:hover {
  color: green;
}
.blog-footer {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.95);
  bottom: 0;
  margin: 0;
  width: 100%;
}
.footer-copyright {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  padding: 8px;
}
.copyright {
  color: #fff;
  font-size: 12px;
  margin: 0;
}
.comment-content h6 {
  color: #1f1f1f;
  padding: 0;
}
.user-profile {
  width: 40px;
  height: 40px;
  object-fit: cover;
  object-position: center;
  border-radius: 100%;
}
@media only screen and (max-width: 770px) {
  .edit-btn,
  .delete-btn {
    min-width: 30px;
  }
  .comment-delete-btn svg,
  .comment-post-btn svg {
    color: #1f1f1f;
    width: 20px;
    height: 20px;
  }
  .edit-btn svg,
  .delete-btn svg {
    color: #1f1f1f;
    width: 20px;
    height: 20px;
  }
  .user-profile {
    width: 20px;
    height: 20px;
    object-fit: cover;
    object-position: center;
    border-radius: 100%;
  }
  .error-container {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    text-align: center;
    margin-top: 10rem;
    width: 95%;
    font-family: "Cabin", sans-serif;
    font-weight: 600;
    margin: 0;
    padding: 0;
  }
  .updated {
    display: flex;
    width: 100%;
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;
    font-size: 10px;
    column-gap: 0.5rem;
    font-weight: 400;
    padding: 0;
    margin: 0;
  }
  svg {
    width: 10px;
  }
  .post-header h2 {
    font-size: 32px;
    font-weight: 600;
    width: 100%;
    margin: 0;
  }

  /* TEXT MARKUP */
  .post-content {
    display: flex;
    justify-content: center;
    align-items: flex-start;
    flex-direction: column;
    width: 100%;
    padding: 0;
    row-gap: 1rem;
  }
  .post-content h3 {
    font-size: 28px;
  }
  .post-markup {
    display: flex;
    justify-content: center;
    align-items: flex-start;
    flex-direction: column;
    width: 100%;
    padding: 0;
    margin: 0;
    row-gap: 0;
  }
  .post-markup h1 {
    font-size: 24px;
    font-weight: 600;
    width: 100%;
    margin: 0;
  }
  .post-markup h2 {
    font-size: 22px;
    font-weight: 600;
    width: 100%;
    margin: 0;
  }
  .post-markup h3 {
    font-size: 20px;
    font-weight: 600;
    width: 100%;
    margin: 0;
  }
  .post-markup h4 {
    font-size: 18px;
    font-weight: 600;
    width: 100%;
    margin: 0;
  }
  .post-markup h5 {
    font-size: 17px;
    font-weight: 600;
    width: 100%;
    margin: 0;
  }
  .post-markup h6 {
    font-size: 16px;
    font-weight: 600;
    width: 100%;
    margin: 0;
  }
  .post-markup p {
    font-size: 14px;
    margin: 0;
  }
  .post-wrapper {
    display: flex;
    flex-direction: column;
    row-gap: 1rem;
    width: 100%;
    margin: 3.5rem 0;
    justify-content: center;
    align-items: center;
  }
  .post-header {
    display: flex;
    justify-content: center;
    align-items: flex-start;
    flex-direction: column;
    width: 100%;
    margin: 0;
  }
  .post-content {
    display: flex;
    justify-content: center;
    align-items: flex-start;
    flex-direction: column;
    width: 100%;
    padding: 0;
    row-gap: 1rem;
  }
  .post-image {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .post-image img {
    width: 100%;
    height: 250px;
    object-fit: cover;
    object-position: center;
  }
  .post-container {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 95%;
    row-gap: 1rem;
  }
  .action-buttons {
    display: flex;
    width: 100%;
    justify-content: flex-start;
    align-items: center;
    column-gap: 1rem;
  }
  .comments-wrapper {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: flex-start;
    flex-direction: column;
    row-gap: 1rem;
    padding: 0;
  }
  .comment-placeholder h6 {
    font-size: 12px;
    font-weight: 100;
  }
  .comments {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-direction: column;
    row-gap: 0.5rem;
    width: 100%;
    margin: 0;
  }
  .comment-info {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-direction: row;
    width: 100%;
  }
  .comment-container {
    width: 95%;
    display: flex;
    justify-content: center;
    align-items: flex-start;
    flex-direction: column;
    row-gap: 1rem;
    margin: 2rem 0;
    padding: 0;
  }
  .text-container {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    row-gap: 1rem;
    padding: 0;
    width: 100%;
  }
  #comments {
    width: 100%;
  }
  .comment-btn {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    flex-direction: row;
    padding: 0;
    margin: 0;
    width: 100%;
  }
  .comment-content p {
    font-family: "Cabin", sans-serif;
    font-style: normal;
    font-size: 12px;
  }
  .comment-content h6 {
    font-size: 15px;
  }
}
</style>
