<template>
    <div class="row">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">
                    Hasil ujian
                </div>
                <div class="card-body">
                    <div class="row">
                        <div class="col-sm-5">
                            <h4 id="traffic" class="card-title mb-0">Hasil ujian</h4>
                            <div class="small text-muted">Filter dengan jadwal</div>
                        </div>
                        <div class="d-none d-md-block col-sm-7">
                            <button :disabled="isLoading" v-if="jadwal != 0" class="btn float-right btn-success btn-sm mx-1" @click="$bvModal.show('modal-scoped-download-hasil-ujian')">
                                <i class="flaticon-download"></i>
                                Download hasil ujian
                            </button>
                        </div>
                    </div>
                    <div class="row" v-if="ujians">
                        <div class="col-md-5">
                            <b-form-group
                              label="Ujian"
                              label-cols-sm="6"
                              label-cols-md="4"
                              label-cols-lg="3"
                              label-align-sm="right"
                              label-size="sm"
                            >
                                <v-select label="alias" :options="ujians" v-model="jadwal" :reduce="nama => nama.id"></v-select>
                            </b-form-group>
                        </div>
                    </div>
                    <div class="row" v-if="jurusands">
                        <div class="col-md-5">
                            <b-form-group
                              label="Jurusan"
                              label-cols-sm="6"
                              label-cols-md="4"
                              label-cols-lg="3"
                              label-align-sm="right"
                              label-size="sm"
                            >
                                <v-select label="nama" :options="jurusands" v-model="jurusan_select" :reduce="nama => nama.id"></v-select>
                            </b-form-group>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-5">
                            <b-form-group
                              label="Per page"
                              label-cols-sm="6"
                              label-cols-md="4"
                              label-cols-lg="3"
                              label-align-sm="right"
                              label-size="sm"
                              label-for="perPageSelect"
                            >
                              <b-form-select
                                v-model="perPage"
                                id="perPageSelect"
                                size="sm"
                                :options="pageOptions"
                              ></b-form-select>
                            </b-form-group>
                        </div>
                    </div>
                    <div class="table-responsive-md">
                        <b-table striped hover bordered small :fields="fields" :items="hasils.data" show-empty>
                            <template v-slot:cell(show_details)="row">
                                <b-button size="sm" @click="row.toggleDetails" :variant="row.detailsShowing ? 'danger' : 'info'"><i :class="row.detailsShowing ? 'flaticon-circle' : 'flaticon2-add'" /></b-button>
                            </template>
                            <template v-slot:row-details="row">
                                <b-card>
                                    <table class="table table-bordered">
                                        <tr>
                                            <td width="200px">Listening</td>
                                            <td>
                                                Salah {{ row.item.jumlah_salah_listening }} : Benar {{ row.item.jumlah_benar_listening }}
                                            </td>
                                        </tr>
                                        <tr>
                                            <td>Pilihan ganda</td>
                                            <td>
                                                Salah {{ row.item.jumlah_salah }} : Benar {{ row.item.jumlah_benar }}
                                            </td>
                                        </tr>
                                        <tr>
                                            <td>Pilihan ganda komplek</td>
                                            <td>
                                                Salah {{ row.item.jumlah_salah_complek }} : Benar {{ row.item.jumlah_benar_complek }}
                                            </td>
                                        </tr>
                                        <tr>
                                            <td>Isian singkat</td>
                                            <td>
                                                Salah {{ row.item.jumlah_salah_isian_singkat }} : Benar {{ row.item.jumlah_benar_isian_singkat }}
                                            </td>
                                        </tr>
                                        <tr>
                                            <td>Menjodohkan</td>
                                            <td>
                                                Salah {{ row.item.jumlah_salah_menjodohkan }} : Benar {{ row.item.jumlah_benar_menjodohkan }}
                                            </td>
                                        </tr>
                                        <tr>
                                            <td>Kosong</td>
                                            <td v-text="row.item.tidak_diisi"></td>
                                        </tr>
                                        <tr>
                                            <td>Point esay</td>
                                            <td v-text="row.item.point_esay"></td>
                                        </tr>
                                        <tr>
                                            <td>Hasil akhir</td>
                                            <td v-text="row.item.hasil"></td>
                                        </tr>
                                    </table>
                                </b-card>
                            </template>
                            <template v-slot:cell(action)="row">
                                <b-button size="sm" variant="primary" :to="{ name: 'kelola.hasil.ujian.siswa', params: { id: row.item.id }}">
                                    Detail jawaban
                                </b-button>
                            </template>
                        </b-table>
                    </div>
                    <div class="row" v-if="typeof hasils.data != 'undefined'">
                        <div class="col-md-6">
                            <p>{{ hasils.data.length }} data dari {{ hasils.total }} total hasil ujian</p>
                        </div>
                        <div class="col-md-6">
                            <div class="float-right">
                                <b-pagination
                                   size="sm"
                                    v-model="page"
                                    :total-rows="hasils.total"
                                    :per-page="hasils.per_page"
                                    aria-controls="products"
                                    v-if="hasils.data && hasils.data.length > 0"
                                    ></b-pagination>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <b-modal id="modal-scoped-download-hasil-ujian" hide-header>
		    <div class="form-group">
                <label>Jurusan</label>
                <v-select label="nama" :options="jurusans" multiple v-model="jurusan_download" :reduce="nama => nama.id"></v-select>
            </div>
            <template v-slot:modal-footer="{ cancel }">
		      <b-button size="sm" variant="primary" @click="download" :disabled="isLoading || jurusan_download == 0 || jurusan_download == '' || jurusan_download == null">
		        {{ isLoading ? 'Processing...' : 'Download' }}
		      </b-button>
		      <b-button size="sm" variant="secondary" @click="cancel()" :disabled="isLoading">
		        Cancel
		      </b-button>
		    </template>
		</b-modal>
    </div>
</template>
<script>
import { mapActions, mapState, mapGetters } from 'vuex'
import { successToas, errorToas} from '@/entities/notif'
import vSelect from 'vue-select'
import 'vue-select/dist/vue-select.css';
import downloadExcel from 'vue-json-excel';

export default {
    name: 'DataHasilUjian',
    components: {
        vSelect,
        downloadExcel
    },
    data() {
        return {
            jadwal: 0,
            perPage: 40,
            pageOptions: [40, 100, 200],
            fields: [
                { key: 'show_details', label: 'Detail' },
                { key: 'peserta.no_ujian', label: 'No ujian' },
                { key: 'peserta.nama', label: 'Nama peserta' },
                { key: 'action', label: 'Aksi' },
            ],
            json_fields: {
                'No ujian' : 'peserta.no_ujian',
                'Nama peserta' : 'peserta.nama',
                'PG Salah' : 'jumlah_salah',
                'PG Benar' : 'jumlah_benar',
                'Listening Salah' : 'jumlah_salah_listening',
                'Listening Benar' : 'jumlah_benar_listening',
                'Kosong' : 'tidak_diisi',
                'Point esay' : 'point_esay',
                'Hasil akhir' : 'hasil'
            },
            jurusan_download: 0,
            jurusan_select: 0
        }
    },
    computed: {
        ...mapGetters(['isLoading']),
        ...mapState('ujian', { 
            ujians: state => state.ujianAll,
            hasils: state => state.hasilUjian
        }),
        ...mapState('jurusan', { jurusans: state => state.all_jurusan }),
        page: {
            get() {
                return this.$store.state.ujian.page_hasil
            },
            set(val) {
                this.$store.commit('ujian/SET_PAGE_HASIL', val)
            }
        },
        jurusands() {
            if (this.jurusans == '') {
                return []
            }
            const jurusan = this.jurusans.map((item) => item);
            jurusan.push({id: 0, nama: 'Semua'})
            return jurusan
        }
    },
    methods: {
        ...mapActions('ujian',['getAllUjians', 'getHasilUjian','getLinkExcelHasilUjian']),
        ...mapActions('jurusan',['allJurusan']),
        async download() {
            if(!this.jadwal) {
                this.$swal({
                  title: 'Hei!!',
                  text: "Pilih ujian terlebih dahulu",
                  icon: 'error',
                })
                return;
            }

            try {
                let provider = await this.getLinkExcelHasilUjian({
                    ujian: this.jadwal,
                    jurusan: this.jurusan_download.join(',')
                })
                window.open(provider.data, '_self')
            } catch (error) {
                this.$bvToast.toast(error.message, errorToas())
            }
        }
    },
    async created() {
        try {
            await this.getAllUjians()
        } catch (error) {
            this.$bvToast.toast(error.message, errorToas())
        }
        this.allJurusan()
    },
    watch: {
        async jadwal(val) {
            if(val != 0) {
                try {
                    await this.getHasilUjian({ 
                        id: val, 
                        jurusan: this.jurusan_select != '' ? this.jurusan_select : 0,
                        perPage: this.perPage 
                    })
                } catch (error) {
                    this.$bvToast.toast(error.message, errorToas())
                }
            }
        },
        async jurusan_select(v) {
            if(v !== '' && v != null) {
                try {
                    await this.getHasilUjian({ 
                        id: this.jadwal, 
                        jurusan: v,
                        perPage: this.perPage 
                    })
                } catch (error) {
                    this.$bvToast.toast(error.message, errorToas())
                }
            }
        },
        page() {
            this.getHasilUjian({
                id: this.jadwal,
                jurusan: this.jurusan_select,
                perPage: this.perPage
            })
        }
    }
}
</script>