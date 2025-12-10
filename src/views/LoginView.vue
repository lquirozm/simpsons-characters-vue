<script setup>


</script>
<template>
  <div class="flex flex-col items-center justify-center h-screen">
    <h1 class="text-3xl mb-6">Login Page</h1>
    <form class="flex flex-col gap-4 w-80">
      <input type="text" placeholder="Username" class="border p-2 rounded"/>
      <input type="password" placeholder="Password" class="border p-2 rounded"/>
      <el-button type="submit" class="bg-blue-500 text-white p-2 rounded hover:bg-blue-600">Login</el-button>
    </form>
  </div>
</template>
