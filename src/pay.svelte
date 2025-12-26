<script>
  export let token = ''; // التوكن القادم من App.svelte

  async function pay() {
    const myApi = window.my;
    if (!myApi) {
      alert('بيئة الدفع غير جاهزة، تأكد أن miniapp تعمل داخل المتصفح الصحيح.');
      return;
    }

    if (!token) {
      myApi.alert({
        content: 'لا يوجد توكن دفع صالح، أعد تسجيل الدخول.'
      });
      return;
    }

    try {
      const res = await fetch('https://its.mouamle.space/api/payment', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          Authorization: token
        }
      });

      const data = await res.json();

      if (!data?.url) {
        myApi.alert({
          content: 'لم نحصل على رابط الدفع من السيرفر.'
        });
        return;
      }

      myApi.tradePay({
        paymentUrl: data.url,
        success: () => {
          myApi.alert({
            content: 'Payment successful'
          });
        },
        fail: () => {
          myApi.alert({
            content: 'Payment failed'
          });
        }
      });
    } catch (err) {
      myApi.alert({
        content: 'Payment failed'
      });
    }
  }
</script>

<main class="min-h-screen bg-gray-100 flex items-center justify-center px-4">
  <button
    class="bg-blue-600 text-white py-2 px-6 rounded-lg font-semibold hover:bg-blue-700 transition"
    on:click={pay}
  >
    ادفع الآن
  </button>
</main>

<style>
  /* ...existing code... */
</style>
