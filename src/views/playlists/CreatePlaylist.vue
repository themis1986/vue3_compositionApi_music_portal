<template>
  <form @submit.prevent="handleSubmit">
    <h4>Create New Playlist</h4>
    <input type="text" required placeholder="Playlist title" v-model="title" />
    <textarea
      required
      placeholder="Playlist description..."
      v-model="description"
    ></textarea>
    <!-- Upload playlist image -->
    <label>Upload playlist cover image</label>
    <input type="file" @change="handleChange" />
    <div class="error">{{ fileError }}</div>
    <button v-if="!isPending">Create</button>
    <button v-else disabled>Saving...</button>
  </form>
</template>

<script>
import { ref } from "vue";
import useStorage from "@/composables/useStorage";
import useCollection from "@/composables/useCollection";
import getUser from "@/composables/getUser";
import { timestamp } from "@/firebase/config";
import { useRouter } from "vue-router";

export default {
  setup() {
    const title = ref("");
    const description = ref("");
    const file = ref(null);
    const fileError = ref(null);
    const isPending = ref(false);

    const router = useRouter();

    const { url, filePath, uploadImage } = useStorage();
    const { user } = getUser();
    const { error, addDoc } = useCollection("playlists");

    const handleSubmit = async () => {
      if (file.value) {
        isPending.value = true;
        // console.log(title.value, description.value, file.value);
        await uploadImage(file.value);
        // console.log("image uploaded, url:", url.value);
        const response = await addDoc({
          title: title.value,
          description: description.value,
          userId: user.value.uid,
          username: user.value.displayName,
          coverUrl: url.value,
          imageFilePath: filePath.value,
          songs: [],
          createdAt: timestamp(),
        });
        isPending.value = false;
        if (!error.value) {
          console.log("playlist added");
          router.push({ name: "PlaylistDetails", params: { id: response.id } });
        }
      }
    };

    // allowed file types
    const types = ["image/png", "image/jpeg"];

    const handleChange = (evt) => {
      const selected = evt.target.files[0];
      // console.log(selected);
      if (selected && types.includes(selected.type)) {
        file.value = selected;
        fileError.value = null;
      } else {
        file.value = null;
        fileError.value = "Please select an image file (png or jpg)";
      }
    };

    return {
      title,
      description,
      fileError,
      isPending,
      handleSubmit,
      handleChange,
    };
  },
};
</script>

<style>
input[type="file"] {
  border: 0;
  padding: 0;
}

label {
  font-size: 12px;
  display: block;
  margin-top: 30px;
}
button {
  margin-top: 20px;
}
</style>
