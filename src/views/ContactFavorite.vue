<template>
    <div class="page row">
        <div class="col-md-10">
            <InputSearch v-model="searchText" />
        </div>
        <div class="mt-3 col-md-6">
            <h4>
                Danh bạ yêu thích
                <i class="fas fa-address-book"></i>
            </h4>
            <ContactList v-if="filteredFavoriteContactsCount > 0" :contacts="filteredFavoriteContacts"
                v-model:activeIndex="activeIndex" />
            <p v-else>Không có liên hệ nào.</p>
        </div>

        <div class="mt-3 col-md-6">
            <div v-if="activeContact">
                <h4>
                    Chi tiết Liên hệ
                    <i class="fas fa-address-card"></i>
                </h4>
                <ContactCard :contact="activeContact" />
                <router-link :to="{
                name: 'contact.edit',
                params: { id: activeContact._id },
            }">
                    <span class="mt-2 badge badge-warning">
                        <i class="fas fa-edit"></i>
                        Hiệu chỉnh
                    </span>
                </router-link>
            </div>
        </div>
    </div>
</template>

<script>
import ContactCard from "@/components/ContactCard.vue";
import InputSearch from "@/components/InputSearch.vue";
import ContactList from "@/components/ContactList.vue";
import ContactService from "@/services/contact.service";
export default {
    components: {
        ContactCard,
        InputSearch,
        ContactList,
    },
    data() {
        return {
            contacts: [],
            activeIndex: -1,
            searchText: "",
        };
    },
    watch: {
        searchText() {
            this.activeIndex = -1;
        },
    },
    computed: {
        // Chuyển các đối tượng contact thành chuỗi
        contactStrings() {
            return this.contacts.map((contact) => {
                const { name, email, address, phone } = contact;
                return [name, email, address, phone].join("");
            });
        },
        
        activeContact() {
            if (this.activeIndex < 0) return null;
            return this.filteredFavoriteContacts[this.activeIndex];
        },
        filteredFavoriteContacts() {
            if (!this.searchText) return this.contacts;
            return this.contacts.filter((_contact, index) =>
                this.contactStrings[index].includes(this.searchText) && _contact.favorite
            );
        },
        filteredFavoriteContactsCount() {
            return this.filteredFavoriteContacts.length;
        },
    },
    methods: {
        async retrieveContacts() {
            try {
                this.contacts = await ContactService.findFavorite();
            } catch (error) {
                console.log(error);
            }
        },
        refreshList() {
            this.retrieveContacts();
            this.activeIndex = -1;
        },
        
    },
    mounted() {
        this.refreshList();
    },
};
</script>

<style scoped>
.page {
    text-align: left;
    max-width: 750px;
}
</style>