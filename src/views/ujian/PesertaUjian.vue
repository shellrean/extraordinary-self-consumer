<template>
	<div class="row">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">
                    Peserta ujian
                </div>
                <div class="card-body">
                    <div class="row">
                        <div class="col-sm-5">
                            <h4 id="traffic" class="card-title mb-0">Status ujian peserta</h4>
                            <div class="small text-muted">Reset dan force close ujian peserta</div>
                        </div>
                        <div class="d-md-block col-sm-7">
                            <b-button variant="primary" class="float-right" size="sm" @click="refresh" :disabled="isLoading" v-if="jadwal != 0 && jadwal != ''">
                                <i class="flaticon-refresh"></i> Refresh data
                            </b-button>
                        </div>
                    </div>
                    <br>
                    <div class="row">
                        <div class="col-md-5">
                            <b-form-group
                              label="Filter"
                              label-cols-sm="3"
                              label-align-sm="right"
                              label-size="sm"
                              label-for="filterInput"
                            >
                              <b-input-group size="sm">
                                <b-form-input
                                  v-model="search"
                                  type="search"
                                  id="filterInput"
                                  placeholder="Cari no ujian"
                                ></b-form-input>
                                <b-input-group-append>
                                  <b-button :disabled="!search" @click="search = ''">Clear</b-button>
                                </b-input-group-append>
                              </b-input-group>
                            </b-form-group>
                            <b-form-group
                              label="Jadwal ujian"
                              label-cols-sm="6"
                              label-cols-md="4"
                              label-cols-lg="3"
                              label-align-sm="right"
                              label-size="sm"
                              label-for="perPageSelect"
                            >
                              <select class="form-control form-control-sm" v-model="jadwal">
                                  <option :value="jadwal.id" v-for="jadwal in jadwals" :key="jadwal.id">{{ jadwal.alias }}</option>
                              </select>
                            </b-form-group>
                        </div>
                    </div>
                    <div class="table-responsive-md">
                        <b-table striped hover bordered small :fields="fields" :items="filteredList" show-empty>
                            <template v-slot:cell(show_details)="row">
                                <b-button size="sm" @click="row.toggleDetails" :variant="row.detailsShowing ? 'danger' : 'info'"><i :class="row.detailsShowing ? 'flaticon-circle' : 'flaticon2-add'" /></b-button>
                            </template>
                            <template v-slot:row-details="row">
                                <b-card>
                                    <b-input-group size="sm">
                                        <b-form-input
                                        placeholder="Waktu dalam menit"
                                        v-model="row.item.addTime"
                                        ></b-form-input>
                                        <b-input-group-append>
                                        <b-button @click="addMoreTime(row.item.addTime, row.item.peserta.id); row.item.addTime = ''" :disabled="isLoading">Tambah</b-button>
                                        </b-input-group-append>
                                    </b-input-group>
                                </b-card>
                            </template>
                            <template v-slot:cell(status)="row">
                                {{ row.item.status }}
                                <b-button variant="danger" :disabled="isLoading" class="mr-1" size="sm" @click="forceClose(row.item.peserta_id)" 
                                v-if="row.item.status_ujian == 3">
                                    Force close
                                </b-button>
                                <b-button variant="outline-danger" :disabled="isLoading" size="sm" @click="resetPeserta(row.item.peserta_id)">
                                    Reset ujian
                                </b-button>
                            </template>
                        </b-table>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import { mapActions, mapState, mapGetters } from 'vuex'
import { successToas, errorToas} from '@/entities/notif'
export default {
    name: 'PesertaUjian',
    data() {
        return {
            fields: [
                { key: 'show_details', label: 'Detail' },
                { key: 'peserta.no_ujian', label: 'No ujian', sortable: true },
                { key: 'peserta.nama', label: 'Nama peserta', sortable: true },
                { key: 'mulai_ujian', label: 'Mulai ujian', sortable: true },
                { key: 'status', label: 'Status', sortable: true }
            ],
            jadwal: 0,
            search: ''
        }
    },
    computed: {
        ...mapGetters(['isLoading']),
        ...mapState('ujian', {
            jadwals: state => state.ujianActive,
            pesertas: state => state.pesertas
        }),
        filteredList() {
            if(this.pesertas) {
                return this.pesertas.filter(post => {
                    return post.peserta.no_ujian.toLowerCase().includes(this.search.toLowerCase())
                })
            }
        }
    },
    methods: {
        ...mapActions('ujian', ['getPesertas', 'resetUjianPeserta', 'selesaiUjianPeserta', 'getUjianActive','addMoreTimeUjianSiswa']),
        async resetPeserta(id) {
            if(this.jadwal == 0) {
                this.$swal({
                  title: 'Hei!!',
                  text: "Pilih jadwal dulu",
                  icon: 'error',
                })
                return
            }
            this.$swal({
                title: 'Informasi',
                text: "Tindakan ini akan menghapus seluruh data ujian siswa, termasuk nilai siswa jika telah di selesaikan sebelumnya.",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#c3c3c3',
                confirmButtonText: 'Iya, Lanjutkan!'
            }).then(async (result) => {
                if (result.value) {
                    try {
                        await this.resetUjianPeserta({id: id, jadwal: this.jadwal})
                        this.getPesertas(this.jadwal)
                        this.$bvToast.toast('Ujian peserta berhasil direset', successToas())
                    } catch (error) {
                        this.$bvToast.toast(error.message, errorToas())
                    }
                }
            })
        },
        async forceClose(id) {
            if(this.jadwal == 0) {
                this.$swal({
                  title: 'Hei!!',
                  text: "Pilih jadwal dulu",
                  icon: 'error',
                })
                return
            }
            this.$swal({
                title: 'Informasi',
                text: "Tindakan ini akan menyelesaikan ujian siswa pada jadwal ini, dan mengenerate nilai.",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#c3c3c3',
                confirmButtonText: 'Iya, Lanjutkan!'
            }).then(async (result) => {
                if (result.value) {
                    try {
                        await this.selesaiUjianPeserta({id: id, jadwal: this.jadwal })
                        this.getPesertas(this.jadwal)
                        this.$bvToast.toast('Ujian peserta berhasil ditutup paksa', successToas())
                    } catch (error) {
                        this.$bvToast.toast(error.message, errorToas())
                    }
                }
            })
        },
        addMoreTime(minutes, peserta_id) {
            this.$swal({
                title: 'Informasi',
                text: "Tindakan ini akan menambah waktu pengerjaan siswa, pastikan waktu yang ditambahkan tidak melebihi waktu ujian.",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#c3c3c3',
                confirmButtonText: 'Iya, Lanjutkan!'
            }).then(async (result) => {
                if (result.value) {
                    try {
                        const network = await this.addMoreTimeUjianSiswa({
                            minutes: parseInt(minutes),
                            peserta_id: peserta_id,
                            jadwal_id: this.jadwal
                        })
                        this.$bvToast.toast(network.message, successToas())
                    } catch (error) {
                        this.$bvToast.toast(error.message, errorToas())
                    }
                }
            })
        },
        refresh() {
            if(this.jadwal == 0) {
                return;
            }
            this.getPesertas(this.jadwal)
        }
    },
    async created() {
        try {
            await this.getUjianActive()
        } catch (error) {
            this.$bvToast.toast(error.message, errorToas())
        }
    },
    watch: {
        async jadwal() {
            if(this.jadwal != 0 || this.jadwal != '') {
                try {
                    await this.getPesertas(this.jadwal)
                } catch (error) {
                    this.$bvToast.toast(error.message, errorToas())
                }
            }
        },
        search() {

        }
    }
}
</script>