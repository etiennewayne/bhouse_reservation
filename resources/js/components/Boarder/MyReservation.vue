<template>
    <div>
        <div class="section">
            <div class="panel">
                <div class="columns">
                    <div class="column is-10 is-offset-1">
                        <div class="is-flex is-justify-content-center mb-2" style="font-size: 20px; font-weight: bold;">MY RESERVATIONS</div>

                        <div class="columns">
                            <div class="column">
                                <b-field label="Page">
                                    <b-select v-model="perPage" @input="setPerPage">
                                        <option value="5">5 per page</option>
                                        <option value="10">10 per page</option>
                                        <option value="15">15 per page</option>
                                        <option value="20">20 per page</option>
                                    </b-select>
                                </b-field>

<!--                                <b-field label="Search">-->
<!--                                    <b-input type="text"-->
<!--                                                v-model="search.lname" placeholder="Search Lastname"-->
<!--                                                @keyup.native.enter="loadAsyncData"/>-->
<!--                                    <p class="control">-->
<!--                                        <b-button type="is-primary" icon-right="account-filter" @click="loadAsyncData"/>-->
<!--                                    </p>-->
<!--                                </b-field>-->
                            </div>
                        </div>


                        <b-table
                            :data="data"
                            :loading="loading"
                            paginated
                            detailed
                            backend-pagination
                            :total="total"
                            :per-page="perPage"
                            @page-change="onPageChange"
                            aria-next-label="Next page"
                            aria-previous-label="Previous page"
                            aria-page-label="Page"
                            aria-current-label="Current page"
                            backend-sorting
                            :default-sort-direction="defaultSortDirection"
                            @sort="onSort">

                            <b-table-column field="user_id" label="ID" v-slot="props">
                                {{ props.row.reservation_id }}
                            </b-table-column>

                            <b-table-column field="boarding_house" label="BHouse" v-slot="props">
                                {{ props.row.rental.boarding_house.bhouse_name }}
                            </b-table-column>

                            <b-table-column field="price" label="Price" sortable v-slot="props">
                                {{ props.row.price }}
                            </b-table-column>

                            <b-table-column field="book_datetime" label="Book Date" v-slot="props">
                                {{ props.row.book_datetime | formatDateTime }}
                            </b-table-column>

                            <b-table-column field="status" label="Status" v-slot="props">
                                <span v-if="props.row.status === 0">PENDING</span>
                                <span v-if="props.row.status === 1">APPROVED</span>
                                <span v-if="props.row.status === 2">CANCELLED</span>
                            </b-table-column>

                            <b-table-column field="aprroved_datetime" label="Approved Date" v-slot="props">
                                <span v-if="props.row.approved_datetime">
                                    {{ props.row.approved_datetime | formatDateTime }}
                                </span>
                            </b-table-column>


                            <b-table-column label="Action" v-slot="props">
                                <b-dropdown aria-role="list" v-if="props.row.status !== 2">
                                    <template #trigger="{ active }">
                                        <b-button
                                            label="..."
                                            type="is-primary"
                                            class="is-small"
                                            :icon-right="active ? 'menu-up' : 'menu-down'" />
                                    </template>

                                    <b-dropdown-item aria-role="listitem"
                                        @click="openModal(props.row.reservation_id)">Upload GCASH Receipt</b-dropdown-item>
                                    <b-dropdown-item aria-role="listitem"
                                        @click="confirmCancel(props.row.reservation_id)"
                                        v-if="props.row.status === 0">Cancel</b-dropdown-item>
                                </b-dropdown>
                            </b-table-column>



                            <template #detail="props">
                                <tr>
                                    <th>ROOM</th>
                                    <th>DESCRIPTION</th>
                                    <th>RECEIPT</th>
                                </tr>

                                <tr>
                                    <td>{{ props.row.rental.rental_name }}</td>
                                    <td>{{ props.row.rental.rental_desc }}</td>
                                    <td>
                                        <button class="button is-outlined is-primary is-small"
                                            v-if="props.row.gcash_receipt_img"
                                            @click="openReceipt(props.row.gcash_receipt_img)">Show receipt</button>
                                    </td>

                                </tr>

                            </template>


                        </b-table>
                    </div><!--close column-->
                </div>

            </div>
        </div><!--section div-->


        <!--modal upload gcash-->
        <b-modal v-model="modalUploadGcash" has-modal-card
            trap-focus
            :width="640"
            aria-role="dialog"
            aria-label="Modal"
            aria-modal>

             <div class="modal-card">
                <header class="modal-card-head">
                    <p class="modal-card-title">UPLOAD GCASH RECEIPT</p>
                    <button
                        type="button"
                        class="delete"
                        @click="modalUploadGcash = false"/>
                </header>

                <section class="modal-card-body">
                    <div class="">
                        <b-field>
                            <b-upload v-model="gcashReceipt" drag-drop>
                                <section class="section">
                                    <div class="content has-text-centered">
                                        <p>
                                            <b-icon
                                                icon="upload"
                                                size="is-large">
                                            </b-icon>
                                        </p>
                                        <p>Drop GCASH receipt here or click to upload</p>
                                    </div>
                                </section>
                            </b-upload>
                        </b-field>

                        <div class="tags" v-if="gcashReceipt.name">
                            <span class="tag is-primary" >
                                {{ gcashReceipt.name }}
                                <button class="delete is-small"
                                    type="button"
                                    @click="gcashReceipt = null">
                                </button>
                            </span>
                        </div>
                    </div>
                </section>
                <footer class="modal-card-foot">
                    <b-button
                        label="Close"
                        @click="modalUploadGcash=false" />
                    <button
                        :class="btnClass"
                        @click="uploadGcashReceipt"
                        type="is-success">Upload</button>
                </footer>
            </div>
        </b-modal>
        <!--close modal upload gcash-->



        <!--modal show receipt -->
        <b-modal v-model="modalShowReceipt" has-modal-card
                 trap-focus
                 :width="640"
                 aria-role="dialog"
                 aria-label="Modal"
                 aria-modal>

            <div class="modal-card">
                <header class="modal-card-head">
                    <p class="modal-card-title">GCASH RECEIPT</p>
                    <button
                        type="button"
                        class="delete"
                        @click="modalShowReceipt = false"/>
                </header>

                <section class="modal-card-body">
                    <div class="">
                        <img :src="`/storage/gcash/${receiptImg}`" />
                    </div>
                </section>
                <footer class="modal-card-foot">
                    <b-button
                        label="Close"
                        @click="modalShowReceipt=false" />
                </footer>
            </div>
        </b-modal>
        <!--close modal show receipt-->

    </div>
</template>

<script>

export default{
    data() {
        return{
            data: [],
            total: 0,
            loading: false,
            sortField: 'user_id',
            sortOrder: 'desc',
            page: 1,
            perPage: 10,
            defaultSortDirection: 'asc',


            global_id : 0,
            gcashReceipt: {},
            receiptImg: '',


            modalUploadGcash: false,
            modalShowReceipt: false,

            fields: {},
            errors: {},

            btnClass: {
                'is-success': true,
                'button': true,
                'is-loading':false,
            },

        }

    },

    methods: {
        /*
        * Load async data
        */
        loadAsyncData() {
            const params = [
                `sort_by=${this.sortField}.${this.sortOrder}`,
                // `key=${this.search.lname}`,
                `perpage=${this.perPage}`,
                `page=${this.page}`
            ].join('&')

            this.loading = true
            axios.get(`/get-my-reservations?${params}`)
                .then(({ data }) => {
                    this.data = [];
                    let currentTotal = data.total
                    if (data.total / this.perPage > 1000) {
                        currentTotal = this.perPage * 1000
                    }

                    this.total = currentTotal
                    data.data.forEach((item) => {
                        //item.release_date = item.release_date ? item.release_date.replace(/-/g, '/') : null
                        this.data.push(item)
                    })
                    this.loading = false
                })
                .catch((error) => {
                    this.data = []
                    this.total = 0
                    this.loading = false
                    throw error
                })
        },
        /*
        * Handle page-change event
        */
        onPageChange(page) {
            this.page = page
            this.loadAsyncData()
        },

        onSort(field, order) {
            this.sortField = field
            this.sortOrder = order
            this.loadAsyncData()
        },

        setPerPage(){
            this.loadAsyncData()
        },

        openModal(resId){
            this.global_id = resId
            this.modalUploadGcash = true;
        },

        openReceipt(img){
            this.modalShowReceipt = true
            this.receiptImg = img
        },

        uploadGcashReceipt(){

            var formData = new FormData();

            formData.append('gcash_receipt_img', this.gcashReceipt ? this.gcashReceipt : '');

            axios.post('/upload-gcash-receipt/' + this.global_id, formData).then(res=>{
                if(res.status.data === 'uploaded'){
                    this.$buefy.dialog.alert({
                        title: 'UPLOADED!',
                        message: 'Successfully uploaded.',
                        type: 'is-success',
                        onConfirm: () => {
                            this.loadAsyncData();
                            this.global_id = 0;
                        }
                    })
                }
            }).catch(err=>{
                if(err.response.status === 422){
                    this.errors = err.response.data.errors;
                }
            })
        },


        //alert box ask for deletion
        confirmCancel(id) {
            this.$buefy.dialog.confirm({
                title: 'DELETE!',
                type: 'is-info',
                message: 'Are you sure you want to cancel reservation?',
                cancelText: 'Close',
                confirmText: 'Yes',
                onConfirm: () => this.cancelSubmit(id)
            });
        },
        //execute delete after confirming
        cancelSubmit(id) {
            axios.post('/cancel-my-reservations/' + id).then(res => {
                if(res.data.status === 'cancelled'){
                    this.loadAsyncData();
                }

            }).catch(err => {
                if (err.response.status === 422) {
                    this.errors = err.response.data.errors;
                }
            });
        },

        clearFields(){
            this.fields = {};
        },


    },

    mounted() {
        this.loadAsyncData();
    }
}
</script>


<style scoped>
    .panel{
        padding: 25px;
    }

</style>
