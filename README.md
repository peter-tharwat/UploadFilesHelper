## Repo to help you handle and upload images & files with best practices (WebP supported)

### Pre requirements

- you have to install some php image extensions

```bash
sudo apt-get install php-imagick php-gd
```

- you have to install webp engine

```bash
sudo apt-get install webp
```

- we are using some packages

```bash
composer require nesbot/carbon
composer require intervention/image
```

- you have to create `hub_files` table

```bash
public function up()
    {
        Schema::create('hub_files', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->unsignedBigInteger('user_id')->nullable();
            $table->foreign('user_id')->references('id')->on("users")->onDelete('cascade'); 
            $table->string('path')->nullable();
            $table->string('name')->index();
            $table->string('extension')->nullable();
            $table->integer('size')->nullable();
            $table->timestamp('used_at')->nullable();
            $table->string('getMimeType')->nullable();
            $table->string('type')->nullable();
            $table->string('type_id')->nullable();
            $table->string('is_main')->nullable();
            $table->string('visibility')->nullable();
            $table->string('bucket_name')->nullable();
            $table->timestamps();
        });
    }
```

- HubFile Model

```bash
class HubFile extends Model
{
    use HasFactory;
    protected $guarded = ['id','created_at','updated_at'];
    
    public function user(){
        return $this->belongsTo(\App\Models\User::class);
    }
    public function getRouteKeyName(){
        return 'name';
    }
    public function get_real_url(){
        return \Storage::disk($this->bucket_name)->url(strtolower($this->visibility).$this->path.$this->name);
    }
    public function get_url(){
        return route('file.link',$this->unique_name);
    }
    public function get_path(){
        return \Storage::disk($this->bucket_name)->path(strtolower($this->visibility).$this->path.$this->name);
    }
    public function get_source(){
        return \Storage::disk($this->bucket_name)->get(strtolower($this->visibility).$this->path.$this->name);
    }
    public function get_size(){
        return \Storage::disk($this->bucket_name)->size(strtolower($this->visibility).$this->path.$this->name);
    }
    public function download(){
        return \Storage::disk($this->bucket_name)->download(strtolower($this->visibility).$this->path.$this->name);
    }
    
}
```

- env file

```bash
CWEBP_ENGINE=/usr/bin/cwebp
```

## Usage

```php
#Upload File
$this->store_file([
    'source'=>$request->file,
    'validation'=>"image",
    'path_to_save'=>'/uploads/users/',
    'type'=>'AVATAR', 
    'user_id'=>auth()->user()->id,
    'resize'=>[500,3000],
    'small_path'=>'small/',
    'visibility'=>'PUBLIC',
    'file_system_type'=>env('FILESYSTEM_DRIVER'),
    'watermark'=>true,
    'compress'=>false,
		'new_extension'=>""//you can set it webp
])['filename'];

#Use File
$this->use_hub_file('file_name','type_id','user_id');
#use multiple files
$uploaded_files=json_decode($request["fileuploader-list-attachment"]);
$attachments=[];foreach($uploaded_files as $uploaded_file)array_push($attachments, $uploaded_file->file);
foreach($attachments as $attachment)
     $this->use_hub_file($attachment, $item->id, auth()->user()->id);

#Remove File
$this->remove_hub_file('file_name');
```
