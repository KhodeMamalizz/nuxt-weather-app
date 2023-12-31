<script setup lang="ts">

// import
    import { useGetLocationData } from '~/composables/useGetLocationData'
    import { useHomeService } from '~/composables/useHomeServices'
    import { useCallApi } from '~/api/useCallApi'
    import Loading from 'vue-loading-overlay'

// state

    document.documentElement.style.overflow = 'hidden'
    
    const { width: windowWidth, height: windowHeight } = useWindowSize()
    const { $gsap: gsap, $Draggable: Draggable, $BusOn, $BusOff } = useNuxtApp();

    //@ts-ignore
    const { getSearchDataFromApi , searchLoading } = useCallApi()
    const { getCurrentLocationData, currentLoading, forecastLoading, getSelectedLocationData } = useGetLocationData()
    const { homeCurrentData, homeSearchData, setHomeSearchData, homeSavedLoctions, removeSavedLocations, clearSearchList } = useHomeService()
    
    const searchModalIsShow = ref<boolean>(false)
    const savedLocationsModalIsShow = ref<boolean>(false)
    const airQualityModalIsShow = ref<boolean>(false)
    const searchQuery = ref<string>('')
    const debouncedearchQuery = refDebounced(searchQuery, 500)


// methods

    const removeLocation = (id: number) => { 
        gsap?.to(`#search-item-${id}`, {
            x: '30rem',
            height: 0,
            opacity: 0,
            duration: 0.5,
        }).then(() => {
            removeSavedLocations(id);
        })
    }

    const refreshAnimtion = () => {
        return new Promise<any>((resolve, reject) => {     
            const refreshTimeline = gsap.timeline({defaults: {duration: 1}})
            refreshTimeline
                .to('.degree', {scale: .75, opacity: 0, ease: 'Power1.easeOut'})
                .to('.hotbar', {y: '100%', ease: 'Bounce.easeOut'})
                .then(() => {
                    resolve(true)
                })
        })
    
    }

    const refresh = () => {
        refreshAnimtion().then(() => {
            getCurrentLocationData()
        })
    }

    const getSelectedLoaction = (cityName: string) => {
        if(homeCurrentData.value.location.name != cityName){
            searchModalIsShow.value = false
            savedLocationsModalIsShow.value = false
            refreshAnimtion().then(() => {
                getSelectedLocationData(cityName)
            })
        }
    }
    
// watch

    watch(debouncedearchQuery, (newValue) => {
        if(newValue == '' || newValue == ' '){
            setHomeSearchData([])
        } else {
            getSearchDataFromApi(newValue)
        }
    })

    watch(savedLocationsModalIsShow, (newValue) => {
        if(newValue){
            setTimeout(() => {
                gsap?.fromTo('.search-items', {opacity: 0}, {opacity: 1, stagger: 0.3, delay: .5, duration: 1})
            }, 100);
        }
    })

    watch(searchModalIsShow, (newValue) => {
        if(!newValue){
            searchQuery.value = ''
            clearSearchList()
        }
    })

    watch(airQualityModalIsShow, (newValue) => {
        if(newValue){
            setTimeout(() => {
                gsap?.fromTo('.air-items', { opacity: 0 }, { opacity: 1, stagger: 0.2})
            }, 100);
        }
    })

// onMounted

    onMounted(() => {
        getCurrentLocationData()

        //@ts-ignore
        $BusOn('open-air-quality-modal' , () => {
            airQualityModalIsShow.value = true
        })
    })

// onUnMounted

    onUnmounted(() => {
        //@ts-ignore
        $BusOff('open-air-quality-modal')
    })

</script>

<template>

    <div class="flex flex-col w-full h-[100vh] parent lg:flex-row-reverse">
        <div class="relative flex items-center justify-center w-full h-full lg:w-1/2">
            <div v-if="currentLoading" class="flex flex-col items-center absolute inset-0 lg:inset-auto top-[22%] lg:top-[50%] lg:-translate-y-1/2">
                <Skeleton
                    class="animate__animated animate__zoomIn"
                    v-for="(i, index) in 4"
                    :key="index"
                    :width="index == 0 ? `13rem` : index == 1 ? '5rem' : index == 2 ? '6.5rem' : index == 3 ? '10rem' : '10rem'" 
                    :height="index == 0 ? `3rem` : index == 1 ? '5rem' : index == 2 ? '2rem' : index == 3 ? '2rem' : '2rem'" 
                    :style="`border-radius: .5rem; margin: 10px 0px;`" 
                />
            </div>

            <div v-else class="flex flex-col items-center absolute w-full lg:w-[20rem] lg:h-[22rem] justify-center lg:backdrop-blur-3xl lg:bg-[#ffffff32] lg:rounded-[2.5rem] top-[22%] lg:top-1/2 lg:-translate-y-1/2 degree">
                <h1 class="text-[2rem] text-white items-center flex gap-2 line-clamp-1">
                    <i class="text-[2rem] fa-solid fa-location-dot"></i>
                    <span class="flex-1">{{ homeCurrentData.location.name ?? 'Undefined' }}</span>
                    <p class="pt-4 text-sm line-clamp-1" v-if="homeCurrentData.location.region">/ {{ homeCurrentData.location.region }}</p>
                </h1>
                <div class="w-1/2 max-w-[15rem] h-[7rem] flex items-center justify-center my-3">
                    <img
                        :src="(homeCurrentData.current.condition.icon).replace('64x64', '128x128')"
                        class="w-[7rem] scale-125 h-[7rem] bg-cover"
                        :alt="homeCurrentData.current.condition.text"
                    />
                </div>
                <p class="text-white text-[2.2rem] -mt-2">{{ homeCurrentData.current.condition.text }}</p>
                <div class="flex gap-x-5">
                    <p class="text-lg text-white">C: {{ homeCurrentData.current.temp_c ?? 0 }}°</p>
                    <p class="text-lg text-white">F: {{ homeCurrentData.current.temp_f ?? 0 }}°F</p>
                </div>
            </div>
            
            <p class="absolute inset-0 flex justify-center text-sm text-white top-10 text-opacity-30"> 
                Last Update : {{ homeCurrentData?.current?.last_updated ?? `--:--:--` }}
            </p>
        </div>

        <client-only>
            <div v-if="currentLoading || forecastLoading" class="relative w-full h-full lg:w-1/2">
                <Skeleton
                    class="duration-100 animate__animated animate__slideInUp rounded-t-[2.5rem] lg:rounded-none bottom-0 lg:inset-0 w-full h-[8rem] lg:h-full"
                    :style="`position: absolute`" 
                />
            </div>
            <div v-else class="relative w-full h-full lg:w-1/2">
                <Hotbar @openModal="openModal" />
            </div>
        </client-only>

        <div class="fixed flex lg:flex-col items-center justify-between w-11/12 lg:w-[5rem] inset-x-0 lg:inset-y-0 lg:inset-x-auto lg:right-5 lg:top-1/2 lg:-translate-y-1/2 mx-auto h-16 lg:h-1/2 px-10 lg:px-1 lg:py-8 border border-[#aa25c933] rounded-full backdrop-blur-md bottom-5 hotbar-buttons">
            <div class="flex items-center justify-start w-1/3 lg:justify-center lg:w-full">
                <button @click="refresh">
                    <lord-icon
                        src="https://cdn.lordicon.com/elzslyvl.json"
                        trigger="hover"
                        colors="primary:#4be1ec,secondary:#cb5eee"
                        style="width:45px;height:45px;">
                    </lord-icon>
                </button>
            </div>

            <div class="flex justify-center w-1/3 lg:w-full">
                <button @click="searchModalIsShow = true" class="flex items-center justify-center transition-all rounded-full shadow-md w-[3rem] h-[3rem] addbtn hover:scale-105">
                    <lord-icon
                        src="https://cdn.lordicon.com/zgogqkqu.json"
                        trigger="hover"
                        colors="primary:#4be1ec,secondary:#cb5eee"
                        style="width:50px;height:50px">
                    </lord-icon>
                </button>
            </div>

            <div class="flex items-center justify-end w-1/3 lg:justify-center lg:w-full">
                <button @click="savedLocationsModalIsShow= true">    
                    <lord-icon
                        src="https://cdn.lordicon.com/dfxesbyu.json"
                        trigger="hover"
                        colors="primary:#4be1ec,secondary:#cb5eee"
                        state="hover-2"
                        style="width:40px;height:40px">
                    </lord-icon>
                </button>
            </div>
        </div>  
    </div>


    <Modal v-if="searchModalIsShow" v-model="searchModalIsShow" title="Find New Locations">

        <div class="mt-4 search-input">
            <div class="w-11/12">
                <input type="text" v-model="searchQuery" placeholder="Search Locations...">
            </div>
            <div class="w-1/12">
                <i class="fa-solid fa-magnifying-glass text-[#ab25c9]"></i>
            </div>
        </div>
        <div class="flex items-center gap-2 text-[.75rem] text-white pl-2 opacity-50" v-if="Object.keys(homeSearchData).length == 0">
            <i class="fa-solid fa-circle-exclamation"></i>
            <p class="pb-0.5">get results by City Name or Country Name</p>
        </div>

        <div class="flex items-center justify-center w-full h-[15rem] loading">
            <Loading
                v-if="searchLoading"
                v-model:active="searchModalIsShow"
                transition="fade"
                color="#ab25c9"
                loader="dots"
                background-color="transparent"
                :can-cancel="true"
                :on-cancel="onCancel"
                :is-full-page="false"
            />

            <p v-else-if="!searchLoading && Object.keys(homeSearchData).length == 0" class="text-white text-[1rem]">
                No results found...!
            </p>

            <div v-else-if="!searchLoading && Object.keys(homeSearchData).length > 0" class="flex flex-col w-full h-full gap-10 mt-3 overflow-scroll pb-[10%]">
                <SearchResultItem 
                    v-for="(data, index) in homeSearchData" 
                    :key="`data-${index}`" 
                    :data="data"
                    @remove="removeLocation"
                    @click="getSelectedLoaction(data.name)"
                />
            </div>
        </div>

    </Modal>

    <Modal v-if="savedLocationsModalIsShow" v-model="savedLocationsModalIsShow" title="Saved Locations">

        <div class="flex items-center justify-center w-full h-[18rem] loading">
            <div v-if="Object.keys(homeSavedLoctions).length > 0" class="flex flex-col w-full h-full gap-10 mt-3 overflow-scroll pb-[10%]" v-dragscroll>
                <SearchResultItem 
                    v-for="(data, index) in homeSavedLoctions" 
                    :id="`search-item-${data.id}`" 
                    :key="`data-${index}-${data.id}`" 
                    :data="data" 
                    @remove="removeLocation"
                    @click="getSelectedLoaction(data.name)"
                />
            </div>

            <p v-else class="text-lg text-white opacity-50 animate__animated animate__zoomIn">
                No saved locations found...!
            </p>

        </div>
        

    </Modal>

    <Modal v-if="airQualityModalIsShow" v-model="airQualityModalIsShow" title="Air includes">

        <div class="flex items-center justify-center w-full h-[18rem]">
            <div class="flex flex-col w-full h-full gap-5 mt-3 overflow-scroll pb-[10%]" v-dragscroll>
                <div 
                    class="flex gap-5 air-items" 
                    v-for="(item, index) in Object.keys(homeCurrentData.current.air_quality)" 
                    :key="`item-${index}`"
                >
                    <div class="bg-[#691a7b33] text-left w-1/2 text-white py-3 px-3 text-lg rounded-xl line-clamp-1 overflow-hidden text-ellipsis whitespace-pre">
                        {{ item }}
                    </div>
                    <div class="bg-[#691a7b33] text-left w-1/2 text-white py-3 px-3 text-lg rounded-xl line-clamp-1 overflow-hidden text-ellipsis whitespace-pre">
                        {{ (homeCurrentData.current.air_quality[item]).toFixed(2) ?? 0 }}
                    </div>
                </div>
            </div>
        </div>

    </Modal>
    
</template>


<style>
.degree {
  transition: opacity 0.3s ease-out; /* Adjust the duration and easing as needed */
  opacity: 1; /* Initially set the opacity to 1 */
}

.addbtn{
    background: #aa25c933;
    box-shadow: 0 1px 10px 0 rgba(109, 31, 135, 1);
    backdrop-filter: blur(8.5px);
    -webkit-backdrop-filter: blur(8px);
    border: 2px solid rgba(255, 255, 255, .3);
}

.search-input {
    @apply flex gap-5 items-center border-2 border-[rgba(137,78,160,1)] focus:ring-2 focus:ring-offset-2 transition-all focus:ring-offset-[trasnparent] ring-[rgba(137,78,160,1)] bg-transparent w-full px-3 py-4 rounded-xl outline-none text-white text-[1rem] tracking-[1px]
}

.search-input input {
    @apply bg-transparent h-full w-full outline-none border-none
}
</style>composables/useGetLocationData