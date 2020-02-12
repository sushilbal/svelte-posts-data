<script>
    import {onMount} from 'svelte';
    import PostForm from '../components/PostForm.svelte';

    const apiBaseUrl ="https://ndb99xkpdk.execute-api.eu-west-2.amazonaws.com/dev";
    let posts=[];
    let postLimit=6;

    let editingPost = {
        body :'',
        title : '',
        id : null
    }

    onMount(async ()=>{
        const res= await fetch(apiBaseUrl+'/posts');
        posts = await res.json();
    })

    function editPost (post){
        console.log(post);
        editingPost = post;
    }

    function deletePost (id){
       if(confirm("Are you sure?")){
           fetch(`${apiBaseUrl}/post/${id}`,{
                method : 'DELETE'
            }).then(res => {
                return res.json();
            }).then(()=>{
                posts = posts.filter( p =>p.id !== id);
            });
            console.log('DeletePost is ', id);
        }
    }

    function addPost({detail : post}){
        if(posts.find(p=> p.id === post.id)){
            const index= posts.findIndex(p=>p.id === post.id);
            let postsUpdate = posts;
            postsUpdate.splice(index,1,post);
            posts=postsUpdate;
        }else{
            posts = [post, ... posts];
        }
        
    }

    function setLimit(){
        
        fetch(`${apiBaseUrl}/posts/${postLimit}`).then(res=>{
            return res.json();
        }).then(postData =>{
            console.log(postData.length);
            posts=postData;
        });
    }


    

</script>
<style>
    .delete-btn{
         color : red !important;
    }
    .card .card-content .card-title {
        margin-bottom :0;
    }
    .card .card-content p.timestamp{
        color : #999;
        margin-bottom : 10px;
    }

</style>
<div class="row">
    <div class="col s6">
        <PostForm on:postCreated={addPost} {editingPost}/>
    </div>
    <div class="col s3" style="margin:50px">
        <p>Limit number of posts</p>
        <input type="number" bind:value={postLimit} />
        <button on:click={setLimit} class="waves-effect waves -lights btn"> Set</button>
    </div>
</div>


<div class="row">
    {#if posts.length === 0}
        <h3>Loading Posts</h3>
    {:else}
         {#each posts as post}
            <div class="col s6">
                <div class="card">
                    <div class="card-content">
                        <p class="card-title">{post.title}</p>
                        <p class="timestamp"> {post.createdAt}</p>
                        <p> {post.body}</p>
                    </div>
                    <div class="card-action">
                        <a href="#" on:click ={()=>editPost(post)}>Edit</a>
                        <a href="#" class="delete-btn" on:click ={()=>deletePost(post.id)}>Delete</a>
                    </div>
                </div>
            </div>
         {/each}
    {/if}
</div>