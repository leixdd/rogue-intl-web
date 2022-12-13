<script lang="ts" setup>
import { ref, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import Swal from 'sweetalert2';

const API_SERVER = ref("https://api.rogueintl.com");

const referrer = ref("");
const account = ref("");
const password = ref("");
const confirm_password = ref("");
const question = ref("父亲姓名");
const answer = ref("");
const captcha = ref("");

const router = useRouter();
const route = useRoute();

const imageSrc = ref("");
const phpsession = ref("");

const sendRegistration = () => {
    //validation
    if (account.value == "") {
        Swal.fire("Please enter your username", "", "error");
        return;
    }
    if (password.value == "") {
        Swal.fire("Please enter your password", "", "error");
        return;
    }
    if (confirm_password.value == "") {
        Swal.fire("Please enter your confirm password", "", "error");
        return;
    }
    if (question.value == "") {
        Swal.fire("Please enter your security question", "", "error");
        return;
    }
    if (answer.value == "") {
        Swal.fire("Please enter your security answer", "", "error");
        return;
    }
    if (captcha.value == "") {
        Swal.fire("Please enter your captcha", "", "error");
        return;
    }

    if(password.value != confirm_password.value){
        Swal.fire("Password and confirm password are not the same", "", "error");
        return;
    }

    fetch(`${API_SERVER.value}/api/v1/rint/register`, {
        method: "POST",
        mode: "cors",
        headers: {
            "Content-Type": "application/json",
        },
        body: JSON.stringify({
            account: account.value,
            password: password.value,
            password2: confirm_password.value,
            question: question.value,
            answer: answer.value,
            vcode: captcha.value,
            d: referrer.value,
            phpsession: phpsession.value,
        }),
    })
        .then((response) => response.json())
        .then((data) => {
            if (data.success) {
                Swal.fire("Registration successful", "", "success");
                account.value = "";
                password.value = "";
                confirm_password.value = "";
                question.value = "父亲姓名";
                answer.value = "";
                captcha.value = "";
                router.push("/");
                fetchCaptch();
                referrer.value = "";
            } else {
                Swal.fire(data.message, "", "error");
            }
        });
    
}

const refreshCaptcha = () => {
    fetchCaptch();
}

const fetchCaptch = () => {
    fetch(`${API_SERVER.value}/api/v1/rint/captcha`, {
        method: "GET",
        mode: "cors",
        headers: {
            "Content-Type": "application/json",
        },
    })
        .then((response) => response.json())
        .then((data) => {
            imageSrc.value = `data:image/png;base64,${data.image}`
            phpsession.value = data.phpsession;
        });
}

onMounted(() => {
    fetchCaptch();
    if (route.params.id) {
        referrer.value = route.params.id.toString() || "";
    }
});
</script>
<template>
    <section class="relative flex flex-wrap lg:h-screen lg:items-center bg-gray-100">
        <div class="w-full px-4 py-12 sm:px-6 sm:py-16 lg:w-1/2 lg:px-8 lg:py-24">
            <div class="mx-auto max-w-lg text-center">
                <h1 class="text-2xl font-bold sm:text-3xl">ROGUE INTERNATIONAL</h1>

                <p class="mt-4 text-gray-500">
                    Join us today!
                </p>
            </div>

            <form @submit.prevent="sendRegistration" class="mx-auto mt-8 mb-0 max-w-md space-y-4">
                <div>
                    <label for="username" class="text-sm font-medium">Username</label>

                    <div class="relative mt-1">
                        <input type="text" id="username" name="username" maxlength="10" required minlength="6" v-model="account"
                            class="w-full rounded-lg border-gray-200 p-4 pr-12 text-sm shadow-sm"
                            placeholder="Enter Username" />
                    </div>
                </div>

                <div>
                    <label for="password" class="text-sm font-medium">Password</label>
                    <div class="relative mt-1">
                        <input type="password" name="password" v-model="password"
                            class="w-full rounded-lg border-gray-200 p-4 pr-12 text-sm shadow-sm"
                            placeholder="Enter Password" />
                    </div>
                </div>

                <div>
                    <label for="confirm_password" class="text-sm font-medium">Confirm Password</label>
                    <div class="relative mt-1">
                        <input type="password" name="confirm_password" v-model="confirm_password"
                            class="w-full rounded-lg border-gray-200 p-4 pr-12 text-sm shadow-sm"
                            placeholder="Enter Confirm Password" />
                    </div>
                </div>

                <div>
                    <label for="question" class="text-sm font-medium">Security Question</label>
                    <div class="relative mt-1">
                        <select name="question" required v-model="question"
                            class="w-full rounded-lg border-gray-200 p-4 pr-12 text-sm shadow-sm">
                            <option value="父亲姓名" selected>Father's name</option>
                            <option value="母亲姓名">Mother's name</option>
                            <option value="最爱的人名称">Favourite person's name</option>
                            <option value="第一款网络游戏">First online game</option>
                        </select>
                    </div>
                </div>

                <div>
                    <label for="answer" class="text-sm font-medium">Security Answer</label>

                    <div class="relative mt-1">
                        <input type="text" id="answer" name="answer" v-model="answer" 
                            class="w-full rounded-lg border-gray-200 p-4 pr-12 text-sm shadow-sm"
                            placeholder="Enter Security Answer" />
                    </div>
                </div>

                <div>
                    <div class="relative">
                        <img :src="imageSrc" alt="captcha" @click="refreshCaptcha"/>
                    </div>
                    <label for="answer" class="text-sm font-medium">Enter Captcha Code</label>

                    <div class="relative mt-1">
                        <input type="text" id="captcha" name="captcha" v-model="captcha"
                            class="w-full rounded-lg border-gray-200 p-4 pr-12 text-sm shadow-sm"
                            placeholder="Enter Security Captcha" />
                    </div>
                </div>



                <div class="flex items-center justify-between">
                    <button type="submit"
                        class="ml-3 inline-block rounded-lg bg-blue-500 px-5 py-3 text-sm font-medium text-white">
                        Register
                    </button>
                </div>
            </form>
        </div>

        <div class="relative h-64 w-full sm:h-96 lg:h-full lg:w-1/2">
            <img alt="Rogue Internation" src="http://www.roguemir.com/img/hero/5_cut.jpg"
                class="absolute inset-0 h-full w-full object-cover" />
        </div>
    </section>

</template>