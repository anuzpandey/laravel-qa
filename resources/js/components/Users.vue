<template>
    <div class="container">
        <div class="row mt-4" v-if="$gate.isAdmin()">
            <div class="col-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">User Lists</h3>

                        <div class="card-tools">
                            <button class="btn btn-primary" @click="newModal">Add New <i
                                    class="fa fa-user-plus fa-fw"></i></button>
                        </div>
                    </div>
                    <!-- /.card-header -->
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover">
                            <thead>
                                <tr>
                                    <th>ID</th>
                                    <th>Name</th>
                                    <th>Email</th>
                                    <th>Type</th>
                                    <th>Registered At</th>
                                    <th>Modify</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="user in users.data" :key="user.id">
                                    <td>{{user.id}}</td>
                                    <td>{{user.name}}</td>
                                    <td>{{user.email}}</td>
                                    <td>{{user.type | capitalize}}</td>
                                    <td>{{user.created_at | properDate}}</td>
                                    <td><span class="tag tag-success">Approved</span></td>
                                    <td>
                                        <a @click="editModal(user)"> <i class="fa fa-edit green ml-2 mr-2"></i></a>
                                        |
                                        <a @click="deleteUser(user.id)"> <i class="fa fa-trash red ml-2"></i></a>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                    <!-- /.card-body -->
                    <div class="card-footer">
                        <pagination :data="users" @pagination-change-page="getResults"></pagination>
                    </div>
                </div>
                <!-- /.card -->
            </div>
        </div>

        <!-- Not Found -->
        <div v-if="!$gate.isAdminOrAuthor()">
            <not-found></not-found>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="exampleModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel"
            aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="exampleModalLabel" v-show="!editMode">Add New User</h5>
                        <h5 class="modal-title" id="exampleModalLabel" v-show="editMode">Update User Info.</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editMode ? updateUser() : createUser()">
                    <div class="modal-body">
                        <!-- Form Here -->
                        <div class="form-group">
                            <input v-model="form.name" type="text" name="name" class="form-control" placeholder="Name"
                                :class="{ 'is-invalid': form.errors.has('name') }">
                            <has-error :form="form" field="name"></has-error>
                        </div>
                        
                        <div class="form-group">
                            <input v-model="form.email" type="email" name="email" class="form-control" placeholder="Email"
                                :class="{ 'is-invalid': form.errors.has('email') }">
                            <has-error :form="form" field="email"></has-error>
                        </div>
                        
                        <div class="form-group">
                            <textarea v-model="form.bio" name="bio" class="form-control" placeholder="Biography"
                                :class="{ 'is-invalid': form.errors.has('bio') }"> </textarea>
                            <has-error :form="form" field="bio"></has-error>
                        </div>

                        <div class="form-group">
                            <select v-model="form.type" name="type" class="form-control"
                                :class="{ 'is-invalid': form.errors.has('type') }">
                                <option value="" disabled>Select User Role</option>    
                                <option value="admin">Admin</option>    
                                <option value="general">General</option>    
                                <option value="author">Author</option>    
                            </select>
                            <has-error :form="form" field="type"></has-error>
                        </div>

                        <div class="form-group">
                            <input v-model="form.password" type="password" name="password" class="form-control" placeholder="Password"
                                :class="{ 'is-invalid': form.errors.has('password') }">
                            <has-error :form="form" field="password"></has-error>
                        </div>

                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                        <button type="submit" v-show="!editMode" class="btn btn-primary">Create</button>
                        <button type="submit" v-show="editMode" class="btn btn-success">Update</button>
                    </div>
                    </form>
                </div>
            </div>
        </div>

    </div>
</template>

<script>
    export default {
        data() {
            return {
                editMode : true,
                users : {},
                form : new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: ''
                })
            }
        },
        methods : {
            newModal() {
                this.editMode = false;
                this.form.reset();
                $('#exampleModal').modal('show');
            },
            editModal(user) {
                this.editMode = true;
                this.form.reset();
                $('#exampleModal').modal('show');
                this.form.fill(user);
            },
            loadUser() {
                if(this.$gate.isAdminOrAuthor()) {
                    axios.get('api/user').then(({data}) => (this.users = data));
                }
            },
            createUser() {
                this.$Progress.start();
                this.form.post('api/user')
                .then(() => {
                    $('#exampleModal').modal('hide');
                    Fire.$emit('AfterCreated');
                    toast.fire({
                        icon: 'success',
                        title: 'User created successfully'
                    });
                    this.$Progress.finish();
                })
                .catch(() => {
                    this.$Progress.fail();
                });
            },
            deleteUser(id) {
                swal.fire({
                title: 'Are you sure?',
                text: "You won't be able to revert this!",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                if (result.value) {
                    //if user clicks confirm button
                    this.form.delete('api/user/'+id)
                    .then(() => {
                        this.$Progress.start();
                        swal.fire(
                            'Deleted!',
                            'Your data has been deleted.',
                            'success'
                        );
                        this.$Progress.finish();
                        Fire.$emit('AfterCreated');
                    })
                    .catch(() => {
                        this.$Progress.fail();
                        swal.fire(
                            'Failed',
                            'There was something wrong.',
                            'warning'
                        );
                    });
                }
                })
            },
            updateUser() {
                this.$Progress.start();
                this.form.put('api/user/'+this.form.id)
                .then(() => {
                    $('#exampleModal').modal('hide');
                    toast.fire({
                        icon: 'success',
                        title: 'Updated Successfully!'
                    });
                    this.$Progress.finish();
                    Fire.$emit('AfterCreated');
                })
                .catch(() => {
                    swal.fire(
                        'Failed',
                        'There was something wrong.',
                        'warning'
                    );
                    this.$Progress.fail();
                });
            },
            getResults(page = 1) {
                axios.get('api/user?page=' + page)
                .then(response => {
                    this.users = response.data;
                })
            }
        },
        mounted() {
            this.loadUser();
            Fire.$on('AfterCreated', () => {
                this.loadUser();
            })
        }
    }
</script>
