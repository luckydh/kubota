<template>
    <div class="gridSearchingContainer" v-show="!searching">
        <div :class="tableClass">
            <table class="grid-view" v-if="data.length > 0 && !hasError">
                <thead>
                    <tr>
                        <th v-for="col in columns" :key="col.field" :class="getClass(col,col.sortable)" v-on:click="sortBy(col.field)">
                            <slot :name="'header_'+col.field">
                                <div v-html="(col.headerFormat || stdFormat) (col.label)">
                                </div>
                            </slot>
                        </th>
                    </tr>
                </thead>
                    <!-- <hr style="position:absolute; border:1px solid red; width:1200px;"> -->
                <tbody>
                    <slot name="grid-content" :items="currentPage">
                        <tr v-for="item in currentPage" :class="getRowClass(item)" :key="tableKey(item)" v-on:click="rowClicked(item)" :title="(toolTipFunc || defaultToolTip)(item)">
                            <td v-for="col in columns" :key="col.field" :class="col.Klass" class="lazy-col">
                                <slot :name="'cell_'+col.field" :item="item">
                                    <div v-html="(col.format || stdFormat)(item[col.field])"></div>
                                </slot>
                            </td>
                        </tr>
                    </slot>
                </tbody>
            </table>
            <slot name="no-content" v-if="!hasError && (!data || data.length === 0)">
                <div class="no-content-message">
                    Nothing to Display
                </div>
                <slot name="error-content" v-if="hasError">
                </slot>
            </slot>
        </div>
    </div>
</template>
<script>
export default {
    name: "Grid",
    data(){
        return { pageEnd:0, dataCopy:[], sortCounter:0, sortField:[] }
    },
    props: {
        data: {
            type: Array,
            required: true
        },
        keyProps: {
            type: Array,
            required: true
        },
        toolTipFunc: {
            type: Array,
            required: false
        },
        searching: {
            type: Boolean,
            default: false
        },
        columns: {
            type: Array,
            required: true
        },
        pageSize: {
            type: Number,
            required: true
        },
        prefix: {
            type: String,
            default: 'table'
        },
        rowClass: {
            type: [Function, String],
            default: 'item'
        },
        filter: {
            type: String,
            default: ''
        },
        hasError: {
            type: Boolean,
            default: false
        },
        sortable: {
            type: Boolean,
            default: false
        },
        selectable: {
            type: Boolean,
            default: false
        },
        headerLocked: {
            type: Boolean,
            default: false
        }
    },
    watch: {
        data(newVal){
            this.pageEnd = Math.min(this.data.length,this.pageSize);
            this.dataCopy = newVal.slice(0);
            console.log(this.data.length,this.pageSize,this.dataCopy,newVal);
        }
    },
    mounted(){
        this.pageEnd = this.pageSize;
        if(this.data && this.data.slice(0));
        this.pageEnd = this.data.length;
    },
    computed: {
        pages(){
            let pageCount = this.pageCount;
            let pages = [];
            for(let i = 0; i < pageCount; i ++) {
                pages.push((i + 1));
            }
            return pages;
        },
        currentPage(){
            return this.filteredData.filter( (el, index) => {
                return index < this.pageEnd;
            })
        },
        tableClass(){
            return this.prefix + '-list';
        },
        filteredData(){
            return this.dataCopy.filter( (el) => {
                return this.inFilter(el);
            });
        }
    },
    methods: {
        stdFormat( val ){
            return val;
        },
        getClass(col){
            let result = col.klass;
            if(this.sortField == col.field){
                if(this.sortCounter == 1){
                    result += 'sort-ascending';
                } else {
                    result += ' sort-descending';
                }
            }
            return result;
        },
        getRowClass(item) {
            let result = this.rowClass;
            if(this.rowClass && this.rowClass.apply) {
                result = this.rowClass(item);
            }
            if(this.selectable) {
                result += "selectable";
                if(item._selected){
                    result += "item-selected"
                }
            }
            return result;
        },
        rowClicked(item){
            if(this.selectable) {
                this.dataCopy.forEach((el) =>{
                    el._selected = false;
                });
                item._selected = true;
                this.dataCopy = this.dataCopy.slice(0);
            }
            this.$emit('rowClick', item);
        },
        inFilter(item){
            let inFilter = true;
            if(this.filter.trim().length > 0) {
                inFilter = false;
                for(let i=0; i < this.columns.length; ++i){
                    if(item[this.columns[i].field].toString.toUpperCase().indexOf(this.filter.toUpperCase().trim()) !== -1){
                        inFilter = true;
                        break;
                    }
                }
            }
            return inFilter;
        },
        defaultToolTip(){
            return "";
        },
        watchScroll(){
            if(this.pageEnd < this.data.length){
                let viewEnd = Math.round(this.$refs.listView.scrollTop + this.$refs.listView.offsetHeight);
                let listHeight = Math.round(this.$$refs.listView.scrollHeight);
                if(viewEnd >= listHeight){
                    this.pageEnd +=this.pageSize;
                }
            }
        },
        changePage(pageNumber){
            this.page = pageNumber;
        },
        tableKey(item){
            let key='';
            for(let i = 0; i< this.keyProps.length; ++i){
                key += item[this.keyProps[i]];
            }
            return key;
        },
        sortBy(field){
            if(this.sortable && this.sortField){
                this.sortCounter++;
            }else{
                this.sortCounter = 1;
                this.sortField = field;
            }
            if(this.sortCounter > 2){
                this.sortCounter = 0;
                this.sortField = '';
                this.dataCopy = this.data.slice(0);
            }else{
                this.dataCopy.sort((a,b) => {
                    let result = 0;
                    if(a[field] > b[field]){
                        result = 1;
                    }else if(a[field] < b[field]){
                        result = -1;
                    }
                    if(this.sortCounter > 1){
                        result *= -1;
                    }
                    return result;
                });
            }
        }
    },
}
</script>

<style scoped>
/* .stickyHeader{
  position: -webkit-sticky;
  position: sticky;
  top: 0;
} */
</style>