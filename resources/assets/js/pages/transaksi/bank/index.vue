<template>
    <!-- TABLE: Transaksi umum -->
    <section class="content">
        <!--<div class="row">
            <tambah/>
        </div>-->
        <div class="row">
            <div class="modal fade" id="modal-generate">
                <div class="modal-dialog middle" >
                    <div class="modal-content">
                        <div class="modal-header">
                        <h4 class="modal-title" style="text-align:center">Generating Excel File</h4>
                        </div>
                        <div class="modal-body">
                            <div class="progress">
                                <div class="progress-bar progress-bar-striped active" role="progressbar" aria-valuenow="100" aria-valuemin="0" aria-valuemax="100" style="width:100%">
                                Please Wait...
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <Tables :empty="empty" :page="page" :currentPage="currentPage" :indexList="indexList" :list="filteredData" :isloading="isloading"/>
            <div class="modal fade" id="modal-edit">
                <div class="modal-dialog" >
                    <div class="modal-content">
                    <form @submit.prevent="submitEdit"  @keydown="form.onKeydown($event)">
                    <div class="modal-header">
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span></button>
                        <h4 class="modal-title">Edit Transaksi</h4>
                    </div>
                    <div class="modal-body">
                        <div class="row">
                            <div class="col-xs-12 col-md-6">
                                <div class="form-group">
                                    <label>Jenis Transaksi</label>
                                    <select class="form-control" v-model="jenisTransaksi" @change="changeJenisTransaksi()">
                                        <option value="1">Transaksi Umum</option>
                                        <option value="2">Transaksi Bank</option>
                                        <option value="3">Transaksi Proyek</option>
                                        <option value="4">Transaksi Unit</option>
                                    </select>
                                </div>
                            </div>
                            <div class="col-xs-12 col-md-6">
                                <div class="form-group">
                                    <label>Kategori</label>
									<input type="text" class="validation" v-model="idKat" required/>
                                     <v-select ref="select2" :inputId="'select2'" :disabled="kategoriDisabled" :value="idKat" @input="changeSelectedKategori" :options="kategoriOptions" :settings="kategoriSetting" @search:focus="maybeLoadKategori"/>
                                </div>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-xs-12 col-md-6">
                                <div class="form-group">
                                    <label>PIC</label>
									<input type="text" class="validation"  v-model="form.idPegawai" required/>
                                    <v-select ref="select3" :inputId="'select3'" @input="changeProyekPeneliti" :value="form.idPegawai" :options="picOptions" :settings="picSetting"  @search:focus="maybeLoadPIC"/>
                                </div>
                            </div>
                            <div class="col-xs-12 col-md-6">
                                <div class="form-group">
                                    <label>Tanggal</label>
									<input type="text" class="validation" v-model="form.tanggal" required/>
                                    <Datepicker v-model="form.tanggal" :format="'dd/MM/yyyy'" :input-class="'form-control bg-datepicker'"/>
                                </div>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-xs-12 col-md-3">
                                <div class="form-group">
                                    <label>Tipe Nominal</label>
                                    <select class="form-control" v-model="form.nominalType" @change="checkNominalChanged">
                                        <option value="d">Debit</option>
                                        <option value="k">Kredit</option>
                                    </select>
                                </div>
                            </div>
                            <div class="col-xs-12 col-md-9">
                                <div class="form-group">
                                    <label>Nominal</label>
                                    <div class="input-group">
                                        <span class="input-group-addon">Rp</span>
                                        <money class="form-control money-text-right" v-model="form.nominal" required/>
                                        <span class="input-group-addon">,00</span>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="form-group">
                            <label>Keterangan</label>
                            <textarea v-model="form.keterangan" class="form-control" rows="3" placeholder="Isikan Keterangan ..."></textarea>
                        </div> 
                        <div class="form-group" v-if="jenisTransaksi!=2 && jenisTransaksi!=null && form.nominalType!=null">
                            <label v-if="form.nominalType == 'd' && form.nominalType!=null">Simpan ke Bank</label>
                            <label v-if="form.nominalType == 'k' && form.nominalType!=null">Ambil dari Bank</label>
                            <div class="input-group">
                                <span class="input-group-addon">
                                <input type="checkbox" v-model="form.isInvolvedBank" @change="checkBoxChanged();">
                                </span>
                                <v-select ref="select" :disabled="!form.isInvolvedBank" v-model="form.idSimpanan" :options="simpananOptions" @search:focus="maybeLoadSimpanan" @change="checkBankChanged"/>
                            </div>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-default pull-left" data-dismiss="modal">Close</button>
                        <v-button :type="'primary'" :loading="form.busy">Save changes</v-button>
                    </div>
                    </form>
                    </div>
                    <!-- /.modal-content -->
                </div>
                <!-- /.modal-dialog -->
            </div>
            <!-- /.modal -->
        </div>
        <div v-if="empty || transaksiList.length <= 5" style="margin-bottom:300px"></div>
    </section>
    <!-- /.box -->
</template>

<script>
    import Tambah from '~/pages/transaksi/bank/tambah'
    import Tables from '~/pages/transaksi/bank/table'
    import Datepicker from 'vuejs-datepicker';
    import Form from 'vform';
    import Cookies from 'js-cookie'
    export default {
        layout: 'default',
        components: {
            Tambah,
            Tables,
            Datepicker
        },
        data: () => ({
            form: new Form({
                idKategori: null,
                idPegawai: {
                    value: null,
                    label: null
                },
                tanggal: null,
                nominalType: null,
                nominal: 0,
                keterangan: null,
                idSimpanan: null,
                idTransaksi: null,
                idProyek: null,
                isKategoriChanged: false,
                isNominalTypeChanged: false,
                isInvolvedBank: false
            }),
            transaksiList : [],
            isloading: false,
            jenisTransaksi: null,
            idKat:null,
			kategoriDisabled: false,
			idPeneliti: [],
            kategoriOptions: [],
            picOptions: [],
            simpananOptions:[],
            kategoriSetting: {
                width:'100%',
                placeholder:'Cari Kategori ...'
            },
            picSetting: {
                width:'100%',
                placeholder:'Cari Nama PIC ...'
            },
            primary: 'primary',
            defaultKat: null,
            defaultBank: null,
            defaultTrans: null,
            defaultNominalType: null,
            page: 0,
            currentPage: 0,
            indexList: [],
            seq: [],
            limit: 10,
            filteredData: [],
            empty: true,
            selectedBank: null,
        }),
        created(){
            Cookies.set('p', 1, { expires: null })
            Cookies.set('t', 2, { expires: null })
        },
        methods: {
            getTransaksiBank(idBank,tahun){
                this.isloading = true;
                let url = '/api/transaksibank/fetch?tahun='+tahun+'&idBank='+idBank
                fetch(url)
                  .then(res => res.json())
                  .then(res => {
                    if(!res.empty){
                        this.transaksiList = res.data;
                        this.isloading = false
                        this.empty = false
                    }
                    else{
                        this.transaksiList = [];
                        this.isloading = false
                        this.empty = true
                    }
                    this.filteredData = this.transaksiList;
                    this.createPagination();
                  })
                  .catch(err => console.log(err));
            },
            submitSearch(searchKey){
                this.filteredData = this.transaksiList.filter(data => data.keterangan.toLowerCase().indexOf(searchKey.toLowerCase(),0) != -1 &&  data.pegawai != '')
                this.createPagination()
            },
            createPagination(){
                this.page = Math.ceil(this.filteredData.length/this.limit);
                this.seq = Array.from(new Array(this.filteredData.length),(val,index)=>index);
                this.indexList = []
                for(var i = 0; i<this.page; i++){
                    this.indexList.push(this.seq.splice(0,this.limit))
                }
                this.currentPage = this.page-1
            },
            editModalShow(i){
                let url = '/api/transaksiedit/showedittrans?idTransaksi='+
                          this.transaksiList[i].id_transaksi+'&jenisTransaksi='+
                          this.transaksiList[i].jenis_transaksi
                console.log(url)
                this.form.reset()
                this.maybeLoadSimpanan()
				var data = null
                fetch(url)
                  .then(res => res.json())
                  .then(res => {
                    data = res.data
                    console.log(data)
                    if(data != 'empty'){
                        this.jenisTransaksi = data.jenis_transaksi
						this.maybeLoadPIC()
                        this.form.idTransaksi = data.id_transaksi
                        this.defaultTrans = data.jenis_transaksi
						this.form.idPegawai.value = data.id_pegawai
                        this.form.idPegawai.label = data.nama_pegawai
						this.maybeLoadKategori()
                        this.defaultKat = data.id_kat
                        this.form.tanggal = data.tanggal
                        this.form.nominalType = data.tipe_nominal
                        this.form.nominal = data.nominal
                        this.defaultNominalType = data.tipe_nominal
                        this.form.keterangan = data.keterangan
                        this.defaultBank = data.id_bank
                        if(data.id_bank != null){
                            this.form.isInvolvedBank= true;
                            this.form.idSimpanan = {
                                value: data.id_bank,
                                label: data.nama_bank
                            }
                        }else{
                            this.form.isInvolvedBank= false;
                            this.form.idSimpanan = null;
                        }
                    }
                  }).then(()=>{
						this.idKat = {
								value: null,
								label: null
						}
						this.idKat.value = data.id_kat
                        this.idKat.label = data.nama_kat
						this.changeSelectedKategori(this.idKat)
						$("#modal-edit").modal('show')
				  })
                  .catch(err => console.log(err))
            },
            submitEdit(){
                let url = '/api/transaksi/edit'
                let date = new Date(this.form.tanggal)
                this.form.patch(url)
                    .then(({data})=>{
                        this.getTransaksiBank(this.selectedBank,date.getFullYear())
                        $("#modal-edit").modal('hide')
                    })
                    .catch(err => console.log(err));
            },
            deleteAlertShow(i){
                this.form.reset()
                this.form.idTransaksi = this.transaksiList[i].id_transaksi
                this.$swal({
                    title: 'Hapus Transaksi '+this.transaksiList[i].keterangan+'?',
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Hapus',
                    cancelButtonText: 'Batal'
                    }).then((result) => {
                        if (result.value) {
                            this.submitDelete(i)        
                        }
                })   
            },
            submitDelete(i){
                let url = '/api/transaksi/delete'
                let date = new Date(this.transaksiList[i].tanggal)
                this.form.patch(url)
                    .then(({data})=>{
                        this.getTransaksiBank(this.selectedBank,date.getFullYear())
                        this.$swal(
                            'Terhapus!',
                            'Transaksi '+this.transaksiList[i].keterangan+' telah berhasil dihapus.',
                            'success'
                        )
                    })
                    .catch(err => console.log(err));
            },
            changeJenisTransaksi(){
				if(this.jenisTransaksi==3){
					this.kategoriDisabled = true;
				}else{
					this.kategoriDisabled = false;
				}
                this.kategoriOptions=[];
                this.form.idKategori=null;
                this.form.idSimpanan= null;
                this.form.idUnit= null;
                this.form.idProyek= null;
                this.form.isInvolvedBank= false;
				this.idKat = null;
				this.picOptions = [];
				this.form.idPegawai = null;
            },
            changeSelectedKategori(value){
				this.idKat = value
                if(this.idKat != null){
                    console.log(this.defaultTrans + ' ' + this.jenisTransaksi + ' ' + this.defaultKat + ' ' + this.idKat.value)
                    if(this.defaultTrans != this.jenisTransaksi || this.defaultKat != this.idKat.value ){
                        this.form.isKategoriChanged = true
                    }else{
                        this.form.isKategoriChanged = false
                    }
                    if(this.jenisTransaksi==1){
                        this.form.idKategori = this.idKat
                    }else if(this.jenisTransaksi==2){
                        this.form.idSimpanan = this.idKat
                    }else if(this.jenisTransaksi==3){
                        this.form.idProyek = this.idKat
                    }else if(this.jenisTransaksi==4){
						this.form.idUnit = this.idKat
					}
                }
            },
			changeProyekPeneliti(value){
				console.log('change proyek peneliti')
				if(this.jenisTransaksi==3){
					this.kategoriOptions=[];
					this.idKat = null;
					this.form.idProyek= null;
					this.kategoriDisabled = false;
				}
				this.form.idPegawai = value;
				console.log(this.form.idPegawai)
			},
            checkBankChanged(){
                if(this.form.idSimpanan != null){
                    if(this.defaultTrans != this.jenisTransaksi || this.defaultBank != this.form.idSimpanan.value 
                       || this.defaultKat != this.idKat.value){
                        this.form.isKategoriChanged = true
                    }else{
                        this.form.isKategoriChanged = false
                    }
                }
            },
            checkBoxChanged(){
                this.form.idSimpanan = null
                this.form.isKategoriChanged = true
            },
            checkNominalChanged(){
                if(this.defaultNominalType != this.form.nominalType){
                    this.form.isNominalTypeChanged = true
                }else{
                    this.form.isNominalTypeChanged = false
                }
            },
            maybeLoadPIC() {
				if(this.jenisTransaksi==3)
					return this.picOptions.length <= 0 ? this.populatePenelitiOptions() : null
				else
					return this.picOptions.length <= 0 ? this.getAllPegawaiList() : null
            },
            maybeLoadKategori() {
                return this.kategoriOptions.length <= 0 ? this.getAllKategoriList(this.jenisTransaksi) : null
            },
            maybeLoadSimpanan(){
                return this.simpananOptions.length <= 0 ? this.getAllSimpananList() : null
            },
            getAllSimpananList(){
                let url = '/api/transaksibank/getallsimpananlist';
                let self = this
                fetch(url)
                  .then(res => res.json())
                  .then(res => {
                    let data = res.data;
                    for(var i = 0; i < data.length; i++){
                        self.simpananOptions.push({
                            label : data[i]['nama_bank'],
                            value : data[i].id
                        })
                    }
                  })
                  .catch(err => console.log(err));
            },
            getAllPegawaiList(){
                let url = '/api/pegawai/getallpegawailist'
                let self = this
				this.$refs.select3.toggleLoading(true)
                fetch(url)
                  .then(res => res.json())
                  .then(res => {
                    let data = res.data;
                    for(var i = 0; i < data.length; i++){
                        self.picOptions.push({
                            label : data[i].nama,
                            value : data[i].id
                        })
                    }
					this.$refs.select3.toggleLoading(false)
                  })
                  .catch(err => console.log(err));
            },
			populatePenelitiOptions(){
				let url = '/api/transaksiproyek/getallpeneliti';
                let self = this
				this.$refs.select3.toggleLoading(true)
                fetch(url)
                  .then(res => res.json())
                  .then(res => {
                    let data = res.data;
					console.log(data)
                    for(var i = 0; i < data.length; i++){
                        self.picOptions.push({
                            label : data[i].pegawai.nama,
                            value : data[i].pegawai.id
                        })
						self.idPeneliti[data[i].pegawai.id] = data[i].id_peneliti
                    }
					this.$refs.select3.toggleLoading(false)
                  })
                  .catch(err => console.log(err));
		   },
            getAllKategoriList(idTrans){
                console.log(idTrans)
                let url = '';
                var name = '';
                switch (+idTrans){
                    case 1:
                        url = '/api/transaksiumum/getallkategorilist';
                        name = 'nama_kategori';
                        break;
                    case 2:
                        url = '/api/transaksibank/getallsimpananlistnoncash';
                        name = 'nama_bank';
                        break;
                    case 3:
                        var dt = (new Date()).getFullYear();
                        url = '/api/transaksiproyek/getallproyeklist?tahun='+dt+'&idPeneliti='
							  +this.idPeneliti[this.form.idPegawai.value] + '&options=true';
                        name = 'nama_proyek';
                        break;
                    case 4:
                        url = '/api/transaksiunit/getallunitlist';
                        name = 'nama';
                        break;
                    default: 
                        url = '';
                        name = '';
                }
                console.log(url)
                let self = this
                this.$refs.select2.toggleLoading(true)
                fetch(url)
                  .then(res => res.json())
                  .then(res => {
                    let data = res.data;
                    console.log(data)
                    for(var i = 0; i < data.length; i++){
                        self.kategoriOptions.push({
                            label : data[i][name],
                            value : data[i].id
                        })
                    }
                    this.$refs.select2.toggleLoading(false)
                  })
                  .catch(err => console.log(err));
            },
            showGenerate(){
                $("#modal-generate").modal({ keyboard: false, backdrop: 'static' })
                $("#modal-generate").modal('show')
            },
            hideGenerate(){
                $("#modal-generate").modal('hide')
            }
        }
    }
</script>
<style>
  .middle{
    margin: 11% 25%;
  }  
  .validation {
	  position: absolute;
	  width: calc(100% - 1px); height: calc(100% - 1px);
	  border: none;
	  border-radius: 5px;
	  background: none;
	  left: 0%; bottom: 0;
	  z-index: -1;
	  opacity: 0;
	}
        .v-select .dropdown-toggle {
    display: flex !important;
}
.v-select .selected-tag {
    overflow: hidden;
    text-overflow: ellipsis; 
    width: 600%;
}
.v-select input {
    width: 100% !important;
}
.bg-datepicker {
    background-color: #ffffff !important;
}
.money-text-right {
    text-align: right;
}
</style>