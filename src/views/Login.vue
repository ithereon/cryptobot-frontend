<template>
    <div class="flex flex-col items-center justify-center px-6 py-8 mx-auto md:h-screen lg:py-0">
        <h2 class="flex items-center mb-6 text-3xl font-semibold text-gray-900 text-white">
            Sign In
        </h2>
        <span>For your protection, please verify your identity.</span>
        <div class="w-full md:mt-0 sm:max-w-lg xl:p-0">
            <div class="p-6 space-y-4 md:space-y-6 sm:p-8">
                <p v-if="errors.length">
                <ul>
                    <li v-for="error in errors" class="text-red-500">{{ error }}</li>
                </ul>
                </p>
                <form class="space-y-4 md:space-y-6" action="#" @submit="handleSubmit">
                    <div>
                        <label for="email" class="block mb-2 text-sm font-medium text-gray-900 text-white">
                            Email Address <span class="text-red-500">*</span></label>
                        <input type="email" name="email" id="email" v-model="email"
                            class="bg-opacity-25 border border-gray-300 sm:text-sm rounded-lg block w-full p-2.5 bg-gray-700 border-gray-600 placeholder-gray-400 text-white focus:ring-gray-500">
                    </div>
                    <div>
                        <label for="password" class="block mb-2 text-sm font-medium text-gray-900 text-white">Password
                            <span class="text-red-500">*</span></label>
                        <input type="password" name="password" id="password" v-model="password"
                            class="bg-opacity-25 border border-gray-300 sm:text-sm rounded-lg block w-full p-2.5 bg-gray-700 border-gray-600 placeholder-gray-400 text-white focus:ring-gray-500">
                    </div>
                    <div class="flex items-center justify-between">
                        <div class="flex items-start">
                            <div class="flex items-center h-5">
                                <input id="remember" type="checkbox" value=""
                                    class="w-4 h-4 text-green-500 accent-green-600 bg-gray-500 border-gray-300 rounded dark:focus:ring-green-600 dark:ring-offset-gray-800 focus:ring-gray-300 dark:bg-gray-700 dark:border-gray-600">
                            </div>
                            <div class="ml-3 text-sm">
                                <label for="remember" class="text-white dark:text-gray-300">Remember me</label>
                            </div>
                        </div>
                        <a href="#"
                            class="text-sm font-medium text-primary-600 hover:underline dark:text-primary-500">Forgot
                            password?</a>
                    </div>
                    <button type="submit"
                        class="w-full bg-transparent hover:bg-green-500 text-green-700 font-semibold hover:text-white py-2 px-4 border border-green-500 hover:border-transparent rounded-lg">Sign
                        in</button>
                    <p class="text-sm font-light text-primary-600 dark:text-gray-400 text-center">
                        Don’t have an account yet?
                        <router-link to="/register" class="font-medium text-green-500 underline dark:text-green-500">
                            Sign up
                        </router-link>
                    </p>
                </form>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import { AuthPayload } from '@/types';
import { useAuthStore } from '@/stores';
import { useUserService } from '@/shared/userService';
import { useBotStore } from '@/stores/ftbotwrapper';


export default defineComponent({
    name: "Login",
    props: {

    },
    emits: [],
    setup() {
        const errors: any = ref([]);
        const email = ref(null);
        const password = ref(null);

        const errorMessage = ref<string>("");
        const botStore = useBotStore();
        const userService = useUserService(botStore.nextBotId);

        const handleSubmit = (e: any) => {
            e.preventDefault();

            if (email.value && password.value) {
                // axios.post("http://localhost:3003/api/auth/signin", {email: email.value, password: password.value})
                // .then(response => {
                //     console.log(response);
                //     if(response.statusText =="OK" && response.data.accessToken){
                //         localStorage.setItem('user', JSON.stringify('bar'+response.data.accessToken));
                //         window.location.href = 'http://localhost:8080/dashboard';
                //     }

                // })
                // .catch(error => {
                // console.error("There was an error!", error);
                // });
                // return true;
                const authStore = useAuthStore();
                return authStore.login(email.value, password.value)
                    .then((res) => {
                        let data = JSON.parse(JSON.stringify(res));
                        console.log("user:", + data);
                        const auth = ref<AuthPayload>({
                            botName: "Athenea1",
                            url: data.user.api_url_bot1,
                            username: data.user.api_username_bot1,
                            password: data.user.api_password_bot1
                        });
                        userService.login(auth.value)
                            .then(() => {
                                const botId = botStore.nextBotId;
                                botStore.addBot({
                                    botName: auth.value.botName,
                                    botId,
                                    botUrl: auth.value.url,
                                });
                                if (botStore.selectedBot === "") {
                                    console.log(`selecting bot ${botId}`);
                                    botStore.selectBot(botId);
                                }
                                botStore.allRefreshFull();
                            })
                            .catch((error) => {
                                console.error(error);
                                if (error.response && error.response.status === 401) {
                                    errorMessage.value =
                                        "Connected to bot, however Login failed, Username or Password wrong.";
                                }
                                else {
                                    errorMessage.value = `Login failed.
                                                        Please verify that the bot is running, the Bot API is enabled and the URL is reachable.
                                                        You can verify this by navigating to ${auth.value.url}/api/v1/ping to make sure the bot API is reachable`;
                                }
                                console.error(errorMessage.value);
                            });
                    }).catch(error => console.error("There was an error!", error));
            }

            errors.value = [];

            if (!email.value) {
                errors.value.push('Email required.');
            }
            if (!password.value) {
                errors.value.push('Password required.');
            }

            e.preventDefault();
        }
        return {
            email,
            password,
            errors,
            handleSubmit
        }
    },
    components: {}
});
</script>

<style scoped>

</style>