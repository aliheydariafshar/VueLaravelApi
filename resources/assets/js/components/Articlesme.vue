<template>
    <div>
        <h2> Articles </h2>
        <hr>
        <form @submit.prevent="addArticle" class="mb-3">
                <div class="form-group">
                    <input type="text" class="form-control" placeholder="Title..." v-model="article.title">
                </div>
                <div class="form-group">
                    <textarea  class="form-control" placeholder="Body..." v-model="article.body"></textarea>
                </div>
            <button type="submit" class="btn btn-block btn-light">Save</button>
        </form>
        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <li :class="[{disabled: !pagination.prev_page_url}]" class="page-item"><a class="page-link" @click.prevent="fetchArticles(pagination.prev_page_url)" href="#">Previous</a></li>
                <li class="page-item disabled"><a class="page-link text-dark" >Page {{pagination.current_page}} of {{pagination.last_page}}</a></li>
                <li :class="[{disabled: !pagination.next_page_url}]" class="page-item"><a class="page-link" @click.prevent="fetchArticles(pagination.next_page_url)" href="#">Next</a></li>
            </ul>
        </nav>
        <div class="card card-body mb-2" v-for="article in articles" :key="article_id">
            <h3>{{article.title}}</h3>
            <p>{{article.body}}</p>
            <hr>
            <div class="btn-group">
                <button class="btn btn-primary w-50" @click.prevent="editArticle(article)">Edit</button>
                <button class="btn btn-danger w-50" @click.prevent="deleteArticle(article.id)">Delete</button>
            </div>
        </div>
    </div>
</template>
<script>
    import sweetalert from 'sweetalert';
    export default {
        data(){
            return{
                articles:[],
                article:{
                    id:'',
                    title:'',
                    body:''
                },
                article_id:'',
                pagination:{},
                edit:false
            }
        },
        methods: {
            fetchArticles(page_url) {
                let vm = this;
                page_url = page_url || 'api/articles';
                fetch(page_url).then(res => res.json()).then( res => {
                    this.articles = res.data;
                    vm.makepagination(res.meta , res.links);
                })
            },
            makepagination(meta , links){
                let pagination = {
                    current_page  : meta.current_page,
                    last_page     : meta.last_page,
                    next_page_url : links.next,
                    prev_page_url : links.prev,


                }
                this.pagination = pagination;
            },
            deleteArticle(id){
                swal({
                    title: "Are You Sure?",
                    dangerMode:true,
                    buttons:[true,"yes I'm sure"],
                    icon:"warning"
                }).then((willDelete) => {
                    if (willDelete) {
                        fetch(`api/article/${id}`, {
                            method: 'delete'
                        }).then(res => res.json()).then(data => {
                            swal("Well Done!", "Article Removed Successfully", "success");
                            this.fetchArticles();
                        }).catch(err => console.log(err))
                    }
                })

            },
            addArticle(){
                if(this.edit === false){
                    fetch('api/article',{
                        method:'post',
                        body:JSON.stringify(this.article),
                        headers:{
                            'content-type':'application/json'
                        }
                    }).then(res => res.json()).then(data=>{
                        this.article.title = '';
                        this.article.body = '';
                        swal("Well Done!","Article Created Successfuly","success");
                        this.fetchArticles();
                    }).catch(err => console.log(err))
                }else if(this.edit === true){
                    fetch('api/article',{
                        method:'put',
                        body:JSON.stringify(this.article),
                        headers:{
                            'content-type':'application/json'
                        }
                    }).then(res => res.json()).then(data=>{
                        this.article.title = '';
                        this.article.body = '';
                        swal("Well Done!","Article Updated Successfuly","success");
                        this.fetchArticles();
                    }).catch(err => console.log(err))

                }
            },
            editArticle(article){
                this.edit = true;
                this.article.article_id = article.id;
                this.article.id = article.id;
                this.article.title = article.title;
                this.article.body = article.body;


            }
        },
        created() {
                this.fetchArticles();
        },
    }
</script>