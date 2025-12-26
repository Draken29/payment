<script>
  function pay() {
    const myApi = window.my;
    if (!myApi) {
      alert('بيئة الدفع غير جاهزة، تأكد أن miniapp تعمل داخل المتصفح الصحيح.');
      return;
    }

    myApi.tradePay({
      paymentUrl: 'https://www.wallet.com/cashier?orderId=xxxxxxx', // TODO: استبدلها بالـ redirectUrl من السيرفر
      success: (res) => {
        myApi.alert({
          content: JSON.stringify(res),
        });
      },
      fail: (res) => {
        myApi.alert({
          content: JSON.stringify(res),
        });
      }
    });
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
