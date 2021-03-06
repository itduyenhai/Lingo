<script>
import Forms from './forms'
import DefaultTab from './../Tabs/default'
import VendorTab from './../Tabs/vendor'

export default {
    components: {DefaultTab, VendorTab},
    name: 'lingo',
    mixins: [Forms],
    props: [
        'currentLocale',
        'scanForMissingRoute',
        'addNewLocaleRoute',
        'addNewFileRoute',
        'addNewVendorRoute',
        'filesRoute',
        'selectedFileDataRoute',
        'deleteFileRoute',
        'deleteLocaleRoute',
        'saveFileRoute'
    ],
    data() {
        return {
            activeTab: 'default-tab',
            lingoTrans: '',
            new_locale: null,
            new_file: null,
            new_vendor: null,
            localesList: [],
            filesList: [],
            dirsList: [],
            selectedDirName: '',
            selectedFileName: ''
        }
    },
    computed: {
        localeExist() {
            return this.localesList && this.localesList.includes(this.new_locale)
        },
        fileExist() {
            return this.filesList && this.filesList.includes(this.new_file)
        },
        dirExist() {
            return this.dirsList && this.dirsList.includes(this.new_vendor)
        }
    },
    created() {
        this.getLingoTrans()
    },
    mounted() {
        this.preVisited()
    },
    updated() {
        this.reflowTable()
    },
    methods: {
        // main
        preVisited() {
            let ls = JSON.parse(localStorage.getItem('lingo'))
            if (ls) {
                this.activeTab = ls.tab

                setTimeout(() => {
                    EventHub.fire('ls-dir', ls.dir)
                    EventHub.fire('ls-file', {
                        tab: ls.tab,
                        val: ls.file
                    })
                }, 50)

            } else {
                localStorage.setItem('lingo', JSON.stringify({
                    tab: this.activeTab,
                    dir: this.selectedDirName,
                    file: this.selectedFileName
                }))
            }
        },
        getLingoTrans() {
            $.post(this.selectedFileDataRoute, {
                'file_name': 'messages.php',
                'dir_name': 'Lingo'
            }, (data) => {
                if (data.success) {
                    this.lingoTrans = data.message.all
                }
            }).fail(() => {
                this.failedAjax()
            })
        },

        // tabs
        activeTabIs(tab) {
            return this.activeTab == tab
        },
        toggleTab(tab) {
            this.activeTab = tab
        },

        // ph
        placeHolder(val, all = '') {
            return this.vendorCheck()
                ? `lang/vendor/${this.selectedDirName}/${all}${val}`
                : `lang/${all}${val}`
        },
        vendorPH(val) {
            return `lang/vendor/${val}`
        },

        // vendor
        vendorCheck() {
            return this.activeTabIs('vendor-tab') && this.selectedDirName
        },
        newVendor() {
            return this.vendorCheck() && this.filesList.length == 0
        },

        // utils
        updateLs(obj) {
            let oldLs = JSON.parse(localStorage.getItem('lingo')) || {}
            Object.assign(oldLs, obj)
            localStorage.setItem('lingo', JSON.stringify(oldLs))
        },
        resetAll(items) {
            items.forEach((e) => {
                this[e] = ''
            })
        },
        reflowTable() {
            setTimeout(() => {
                $('table').trigger('reflow')
            }, 10)
        },
        trans(key) {
            return this.lingoTrans[key][this.currentLocale]
        },

        // notifs
        showNotif(msg, s = 'success') {

            let title = ''
            let duration = null

            switch (s) {
            case 'danger':
            case 'black':
                title = 'Error'
                break
            case 'warning':
                title = 'Warning'
                duration = 2
                break
            default:
                title = 'Success'
                duration = 3
            }

            EventHub.fire('showNotif', {
                title: title,
                body: msg,
                type: s,
                duration: duration
            })
        },
        failedAjax() {
            this.showNotif(this.trans('ajax_fail') || 'Ajax Call Failed', 'black')
        },
        missingVal(msg = null) {
            this.showNotif(msg || this.trans('no_val') || 'Missing Value', 'warning')
        }
    },
    watch: {
        activeTab(val) {
            this.updateLs({tab : val})
        },
        selectedDirName(val) {
            this.updateLs({dir : val})
        },
        selectedFileName(val) {
            this.updateLs({file : val})
        }
    },
    render() {}
}
</script>
