<script setup lang="ts">

// emits

    const emit = defineEmits(["update:modelValue" , 'closeModal'])

// props

    const props = defineProps({
        modelValue: {
          default: false,
        },
        title: {
            type: String,
            default: ''
        },
        closeOnBlur: {
            type: Boolean,
            default: true
        }
    })


// methods
    const closeToastModal = () => {
        emit('update:modelValue', false)
        emit('closeModal')
    }
    
</script>

<template>
     <div
          class="fixed inset-0 bg-[#000000d0] animate__animated animate__fadeIn duration-75 select-none"
          style="z-index: 9999999999999999999999999999999999999999999999999999999999999999999999999999999999999 !important"
          @click="closeOnBlur ? closeToastModal() : null"
     >
          <div
               class="relative flex items-center justify-center w-full h-full"
               style="z-index: 999999999 !important"
          >
               <div
                    @click.stop
                    class="modal relative flex flex-col min-h-[50vh] max-h-[75vh] overflow-scroll w-11/12 gap-4 px-6 pt-10 md:w-[30rem] shrink-0 rounded-[2.188rem] shadow-md animate__animated animate__fadeInDown"
               >
                    <div class="flex items-center justify-between w-full h-8 pr-2">
                        <div class="flex items-center gap-2 opacity-50">
                            <i class="fa-solid fa-grip-lines-vertical text-[#ab25c9] text-[1.5rem] pb-1"></i>
                            <h2 class="pb-1 text-2xl text-left text-white">{{ title }}</h2>
                        </div>
                        <i class="text-lg text-white fa-solid fa-xmark" @click="closeToastModal"></i>
                    </div>
                    <slot />
               </div>
          </div>
     </div>
</template>

<style>

.modal {
    background: #aa25c933;
    box-shadow: 0 1px 10px 0 rgba(109, 31, 135, 1);
    backdrop-filter: blur(8.5px);
    -webkit-backdrop-filter: blur(8px);
    border: 2px solid rgba(255, 255, 255, .3);
}

</style>