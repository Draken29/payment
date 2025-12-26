<script>
  import { onMount } from 'svelte';
  import Pay from './pay.svelte';

  let authCode = '';
  let myApi; // يمسك كائن my القادم من hylid-bridge
  let isAuthenticated = false; // حالة التحقق
  let paymentToken = '';       // توكن الدفع القادم من السيرفر

  onMount(async () => {
    // تحميل السكربت الخارجي ديناميكياً مرة واحدة
    const script = document.createElement('script');
    script.src = 'https://cdn.marmot-cloud.com/npm/hylid-bridge/2.10.0/index.js';
    script.async = true;

    await new Promise((resolve, reject) => {
      script.onload = resolve;
      script.onerror = reject;
      document.body.appendChild(script);
    });

    // بعد التحميل نتوقع وجود window.my
    if (window.my) {
      myApi = window.my;
    } else {
      console.error('hylid-bridge لم يتم تحميله بشكل صحيح، my غير موجود');
    }
  });

  function authenticate() {
    if (!myApi) {
      alert('النظام غير جاهز بعد، حاول مرة أخرى بعد ثوانٍ.');
      return;
    }

    myApi.getAuthCode({
      scopes: ['auth_base', 'USER_ID'],
      success: (res) => {
        authCode = res.authCode;

        fetch('https://its.mouamle.space/api/auth-with-superQi', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            token: authCode
          })
        })
          .then(res => res.json())
          .then((data) => {
            // نتوقع من الـ API يرجع توكن أو شيء مشابه
            paymentToken = data?.token || '';
            isAuthenticated = true; // تم تسجيل الدخول بنجاح
          })
          .catch(err => {
            let errorDetails = '';
            if (err && typeof err === 'object') {
              errorDetails = JSON.stringify(err, null, 2);
            } else {
              errorDetails = String(err);
            }
            myApi.alert({
              content: 'Error: ' + errorDetails
            });
          });
      },
      fail: (res) => {
        console.log(res.authErrorScopes);
      }
    });
  }

  function copyAuthCode() {
    if (!authCode) return;
    navigator.clipboard.writeText(authCode);
  }
</script>

<div class="h-16 w-full bg-stone-700 flex items-center justify-start px-4 fixed top-0 left-0 z-10">
  <h1 class="text-2xl font-bold text-white">Demo MiniApp</h1>
</div>

<div class="w-full h-screen flex flex-col items-center gap-4 justify-start p-4 mt-16">
  <button class="bg-blue-500 text-white px-4 py-2 rounded-md" on:click={authenticate}>
    Login
  </button>
  <button class="bg-blue-500 text-white px-4 py-2 rounded-md" on:click={copyAuthCode}>
    Copy Auth Code
  </button>
  <p class="text-sm text-gray-500">
    Auth Code: <span>{authCode}</span>
  </p>
</div>

<main class="min-h-screen bg-gray-100 flex items-center justify-center px-4">
  {#if isAuthenticated}
    <Pay token={paymentToken} />
  {:else}
    <p class="text-gray-600 text-center">
      يرجى تسجيل الدخول أولاً لرؤية صفحة الدفع.
    </p>
  {/if}
</main>

<style>
  /* ...existing code... */
</style>
