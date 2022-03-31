<template>
  <div class="home-wrapper">
    <div class="home-container">
      <div class="user-actions">
        <div v-if="currentUser" class="create-post">
          <button
            class="create-btn shadow-sm"
            type="button"
            data-bs-toggle="modal"
            data-bs-target="#updateModal"
          >
            Create new blog
          </button>
        </div>
        <div class="search-actions">
          <div class="search">
            <label for="search">Search</label>
            <input
              type="text"
              v-model="search"
              name="search"
              placeholder="Search blogs..."
              class="search-bar shadow"
            />
          </div>
        </div>
      </div>
      <header class="about-header">
        <h3>Recent Posts</h3>
      </header>
      <div
        class="post-item shadow"
        v-for="post in filteredPosts.slice(0, count)"
        :key="post._id"
      >
        <div class="post-image">
          <img :src="post.main_image" />
        </div>
        <div class="post-content">
          <div class="formatted-date">
            <div class="text-muted updated">
              <div class="user">
                <img :src="post.user_image" class="user-image" />
                <span v-show="!loading && !errorMessage" class="text-muted">
                  {{ post.created_by }}
                </span>
              </div>
              <div class="date">
                <span v-show="!loading && !errorMessage" class="text-muted">
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="20"
                    height="20"
                    fill="currentColor"
                    class="bi bi-calendar text-muted"
                    viewBox="0 0 16 16"
                  >
                    <path
                      d="M3.5 0a.5.5 0 0 1 .5.5V1h8V.5a.5.5 0 0 1 1 0V1h1a2 2 0 0 1 2 2v11a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2V3a2 2 0 0 1 2-2h1V.5a.5.5 0 0 1 .5-.5zM1 4v10a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1V4H1z"
                    />
                  </svg>
                </span>
                <span>
                  {{ moment(post.createdAt).format("MMM DD, YYYY") }}
                </span>
              </div>
              <div class="comments">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="20"
                  height="20"
                  fill="currentColor"
                  class="bi bi-chat text-muted"
                  viewBox="0 0 16 16"
                >
                  <path
                    d="M2.678 11.894a1 1 0 0 1 .287.801 10.97 10.97 0 0 1-.398 2c1.395-.323 2.247-.697 2.634-.893a1 1 0 0 1 .71-.074A8.06 8.06 0 0 0 8 14c3.996 0 7-2.807 7-6 0-3.192-3.004-6-7-6S1 4.808 1 8c0 1.468.617 2.83 1.678 3.894zm-.493 3.905a21.682 21.682 0 0 1-.713.129c-.2.032-.352-.176-.273-.362a9.68 9.68 0 0 0 .244-.637l.003-.01c.248-.72.45-1.548.524-2.319C.743 11.37 0 9.76 0 8c0-3.866 3.582-7 8-7s8 3.134 8 7-3.582 7-8 7a9.06 9.06 0 0 1-2.347-.306c-.52.263-1.639.742-3.468 1.105z"
                  />
                </svg>
                <span
                  v-show="!loading && !errorMessage"
                  class="text-muted"
                  v-if="post.comments"
                >
                  {{ post.comments.length }}
                  <span
                    v-show="
                      post.comments.length >= 0 && post.comments.length != 1
                    "
                    >Comments</span
                  >
                  <span v-show="post.comments.length == 1">Comment</span>
                </span>
              </div>
            </div>
          </div>
          <h3>{{ post.title }}</h3>
          <h4>{{ post.subtitle }}</h4>
          <div class="buttons">
            <router-link :to="{ path: `/post/${post._id}` }">
              <button class="read-btn" type="button">Read More...</button>
            </router-link>
          </div>
        </div>
      </div>
      <div class="post-action" :disabled="loading">
        <button
          class="load-btn shadow"
          @click.prevent="loadMore()"
          v-show="!loading && !errorMessage"
        >
          Load More
        </button>
      </div>
    </div>
    <!-- ERROR MESSAGE -->
    <div
      class="error-container"
      v-if="!loading && errorMessage"
      :innerHTML="errorMessage"
    ></div>
    <!-- LOADER -->
    <div v-if="loading">
      <div class="loading-container">
        <Loader />
      </div>
    </div>
  </div>

  <!-- CREATE MODAL -->
  <!-- <pre>
    <header>
    <h3>It all started 8 years ago</h3>
    <p>
      I never really understood the importance of hard work. It didn't really appeal to me at all.
      Upon completing my schooling, I was constantly bombarded with daily reminders of how essential hard work is.
      But what does it really mean to work hard??
    </p>
    </header>
    <br>
    <article>
      <h3>Working hard is not literal!</h3>
    <p>
      What do I mean by that? When you normally picture a hardworking individual, you sort of visualize someone physically executing hard labour.
      At least thats what I feel. It kind of puts you off because who wants to spend their time and energy doing something so mundane?
      I can assume nobody wants to, right?
      So what is it that we need to understand? 
      In my opinion and experience, working hard is simply devoting oneself to personal development and growth. Understand that if you commit to something, with the promise that you will fulfill it to the utmost of your capabilities,
      then you are obligated to invest every ounce of energy, time and enthusiasm that you can muster into achieving not only what you set out to do, but in meeting the requirements that have been set by those before you.
    </p>
    <p>
      That is what hard work means to me. The insatiable drive to and hunger to conquer.
    </p>
    </article>
  </pre> -->

  <!-- Button trigger modal -->

  <div
    class="modal fade"
    id="updateModal"
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
            Create a new blog post.
          </h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          <form @submit.prevent="addPost()" class="row create-form">
            <div class="col-md-12">
              <label for="main-image" class="form-label mt-1">Image *</label>
              <input
                required
                type="text"
                v-model="post.main_image"
                class="form-control"
                placeholder="Insert image link..."
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
                v-model="post.title"
                class="form-control"
                id="title"
              />
            </div>
            <div class="col-md-12">
              <label for="subtitle" class="form-label">Subtitle *</label>
              <input
                required
                type="text"
                v-model="post.subtitle"
                class="form-control"
                id="subtitle"
              />
            </div>
            <div class="col-md-12">
              <label for="desc" class="form-label">Content *</label>
              <textarea
                required
                type="text"
                v-model="post.desc"
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
                class="submit-btn"
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
</template>

<script>
import PostService from "../services/post.service";
import Loader from "../components/Loader.vue";
import moment from "moment";
const url = "https://final-blog-api.herokuapp.com/posts/";
export default {
  name: "AllPosts",
  components: {
    Loader,
  },
  data() {
    return {
      loading: false,
      posts: [],
      count: 2,
      post: {
        main_image: "",
        title: "",
        subtitle: "",
        desc: ``,
        created_by: "",
      },
      errorMessage: null,
      moment,
      search: "",
    };
  },
  computed: {
    currentUser() {
      return this.$store.state.auth.user;
    },
    filteredPosts() {
      return this.posts.reverse().filter((post) => {
        return post.title.toLowerCase().match(this.search.toLowerCase());
      });
    },
  },
  mounted() {
    this.loading = true;
    PostService.getAllPosts()
      .then((res) => {
        this.posts = res.data;
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
    async addPost() {
      try {
        this.loading = true;
        fetch(`${url}create`, {
          method: "POST",
          body: JSON.stringify({
            main_image: this.post.main_image,
            title: this.post.title,
            subtitle: this.post.subtitle,
            desc: this.post.desc,
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
    loadMore() {
      this.loading = true;
      this.count += 2;
      this.loading = false;
    },
  },
};
</script>
<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Cabin&family=Oswald:wght@600&family=Poppins&display=swap");
.home-wrapper {
  display: flex;
  flex-direction: column;
  width: 100%;
  justify-content: center;
  align-items: center;
  margin: 5rem 0;
}
.about-header {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
}
.about-header h3 {
  font-size: 32px;
  font-weight: 600;
  margin: 0;
}
.about-content {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  width: 60%;
  row-gap: 1rem;
}
.about-content h5,
p {
  text-align: center;
}
.home-container {
  margin: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  row-gap: 3rem;
}
.post-item {
  display: flex;
  width: 50%;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  row-gap: 0.5rem;
}
.post-image {
  width: 100%;
}
img {
  width: 100%;
  max-height: 500px;
  object-fit: cover;
  object-position: center;
}
.post-content {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  flex-direction: column;
  row-gap: 0.5rem;
  width: 100%;
  padding: 20px;
  background-color: #fff;
}
h3 {
  font-size: 32px;
  font-weight: 600;
  margin: 0;
  padding: 0;
}
h4 {
  font-size: 24px;
  margin: 0;
  padding: 0;
}
.updated {
  display: flex;
  width: 100%;
  flex-direction: row;
  justify-content: flex-start;
  align-items: center;
  font-family: "Cabin", sans-serif;
  column-gap: 1rem;
  font-weight: 300;
  margin: 0;
}
.user,
.date,
.comments {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: row;
  gap: 5px;
}
.info-message {
  margin-top: 0.5rem;
}
.info-message h6 {
  margin-bottom: 0;
}
textarea {
  min-height: 264px;
  padding: 10px;
}
.username {
  padding: 0;
}
.formatted-date {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
}
.user-image {
  width: 40px;
  height: 40px;
  object-fit: cover;
  object-position: center;
  border-radius: 100%;
}
.buttons {
  width: 100%;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  padding: 0;
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
  padding: 0 !important;
}
.modal-footer {
  display: flex;
  justify-content: flex-end;
  width: 100%;
}
.submission {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  row-gap: 1rem;
}
.search-actions {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: row;
  width: 100%;
  column-gap: 2rem;
}
.search {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: row;
  column-gap: 1rem;
}
.create-post {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: row;
  column-gap: 1rem;
}
.create-btn {
  min-width: 200px;
  padding: 10px;
  outline: none;
  border: none;
  background: rgba(0, 0, 0, 0.95);
  color: #fff;
  transition: all 0.5s;
}
.create-btn:hover {
  background: blueviolet;
  color: #fff;
}
.load-btn {
  min-width: 200px;
  padding: 10px;
  outline: none;
  border: none;
  background: rgba(0, 0, 0, 0.95);
  color: #fff;
  transition: ease-in-out 500ms;
}
.load-btn:hover {
  background: blueviolet;
  color: #fff;
}
.read-btn {
  min-width: 120px;
  padding: 10px;
  outline: none;
  border: none;
  background: none;
  color: rgba(0, 0, 0, 0.95);
  border-bottom: 2px solid rgba(0, 0, 0, 0.95);
}
.submit-btn {
  min-width: 80px;
  padding: 5px;
  outline: none;
  border: none;
  background: transparent;
  color: #1f1f1f;
  transition: ease-in-out 500ms;
}
.submit-btn:hover {
  color: green;
}
.cancel-btn {
  min-width: 80px;
  padding: 5px;
  outline: none;
  border: none;
  background: transparent;
  color: #1f1f1f;
  transition: ease-in-out 500ms;
}
.cancel-btn:hover {
  color: red;
}
.user-actions {
  display: flex;
  align-items: center;
  justify-content: center;
  row-gap: 2rem;
  flex-direction: column-reverse;
}
.search {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  row-gap: 1rem;
}
.search-bar {
  min-width: 200px;
  padding: 10px;
  border: none;
  transition: all 0.5s;
}
.search-bar:focus {
  transform: scale(1.05);
}
.modal-body {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  width: 100%;
  margin: 0;
  padding: 0;
}
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  width: 98%;
  margin: 0;
  padding: 0;
}
.row {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  row-gap: 0.5rem;
  width: 100%;
  margin: 0;
  padding: 0;
}
.col {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  flex-direction: column;
  row-gap: 0.5rem;
  width: 100%;
  margin: 0;
  padding: 0;
}
.col input {
  width: 100%;
}
@media only screen and (max-width: 770px) {
  .about-header h3 {
    font-size: 25px;
    font-weight: 600;
    margin: 0;
  }
  .post-content {
    padding: 10px;
  }
  .user-image {
    width: 20px;
    height: 20px;
    object-fit: cover;
    object-position: center;
    border-radius: 100%;
  }
  svg {
    width: 10px;
    height: 10px;
  }
  .about-content {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    width: 95%;
    row-gap: 1rem;
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
  .home-wrapper {
    display: flex;
    flex-direction: column;
    width: 100%;
    justify-content: center;
    align-items: center;
    margin-top: 3rem;
    margin-bottom: 5rem;
  }
  .home-container {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 100%;
    row-gap: 3rem;
  }
  .post-item {
    display: flex;
    width: 95%;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    row-gap: 0.5rem;
  }
  h3 {
    font-size: 25px;
    font-weight: 600;
    margin: 0;
    padding: 0;
  }
  h4 {
    font-size: 15px;
    margin: 0;
    padding: 0;
  }
  .updated {
    flex-direction: row;
    font-size: 10px;
    font-weight: 400;
    column-gap: 0.5rem;
  }
  .read-btn {
    width: 60px;
    font-size: 12px;
    padding: 10px;
    outline: none;
    border: none;
    background: none;
    color: rgba(0, 0, 0, 0.95);
    border-bottom: 2px solid rgba(0, 0, 0, 0.95);
  }
  .load-btn {
    min-width: 150px;
    padding: 5px;
    outline: none;
    border: none;
    background: rgba(0, 0, 0, 0.95);
    color: #fff;
    transition: ease-in-out 500ms;
  }
  .create-btn {
    min-width: 150px;
    padding: 5px;
    outline: none;
    border: none;
    background: rgba(0, 0, 0, 0.95);
    color: #fff;
    transition: ease-in-out 500ms;
  }
  .search-bar {
    min-width: 300px;
    padding: 10px;
    border: none;
  }
  img {
    width: 100%;
    height: 250px;
    object-fit: cover;
    object-position: center;
  }
}
</style>
