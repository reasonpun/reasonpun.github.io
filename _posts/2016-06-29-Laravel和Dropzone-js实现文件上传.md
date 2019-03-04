---
layout: post
title: Laravel和Dropzone.js实现文件上传
---

Dropzone是一个最好的免费库，可以通过拖拽实现文件的上传。拥有很多特性和选项，可以通过多种方式定制开发。

在Laravel项目中加入Dropzone对于没有经验的新手来说会有很多麻烦，所以在这里会讲些最靠谱的解决方法。

这篇引导会讲到：

  * 文件自动上传（上传队列自动处理）
  * 在预览页面通过ajax直接删除图片
  * 还可以统计上传的图片数量
  * 保存两种图片尺寸：原始尺寸和icon尺寸
  * Using Image Intervention package for resizing and image encoding
  * 保存图片数据到数据库
  * 服务器端保存唯一的文件名

如果正在找一种上传和编辑图片的最贱的方法，可以checkout[Croppic jQuery plugin](http://www.croppic.net/)。Croppic是一个简单的类似Facebook，Twitter或者LinkedIn的照片属性编辑组件。

> 整个项目可以从 https://github.com/codingo-me/dropzone-laravel-image-upload 获取

实现的效果图如下

![Dropzone Laravel]({{ site.url }}/assets/aravel/dropzoneLaravelImg.png)

## 基本的项目配置

我将会使用Laravel安装器安装一版纯净的Laravel环境 and create environment file with database credentials.

首先从 https://github.com/enyo/dropzone/tree/master/dist 下载最新的js和styles文件。然后放到项目的public/packages/dropzone文件夹下。

前端使用Bootstrap。 把相关文件放到public/packages/bootstrap。

需要用到两个第三方包：

```
......
    "require": {
        "php": ">=5.5.9",
        "laravel/framework": "5.2.*",
        "laravelcollective/html": "5.2.*",
        "intervention/image": "^2.3"
    },
......
```

通过composer install/update安装完成后，需要在同一个文件中添加：

```
Intervention\Image\ImageServiceProvider::class,
Collective\Html\HtmlServiceProvider::class,
```

和facades

```
    'Form'      => Collective\Html\FormFacade::class,
    'HTML'      => Collective\Html\HtmlFacade::class,
    'Image'     => Intervention\Image\Facades\Image::class
```

## 视图

这个引导只包含一个用来上传图片的页面。但是最好能分割成不同的模块：主布局文件和独立的文件；

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Image upload in Laravel 5.2 with Dropzone.js</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    {!! HTML::style('/packages/bootstrap/css/bootstrap.min.css') !!}
    {!! HTML::style('/assets/css/style.css') !!}
    {!! HTML::script('https://code.jquery.com/jquery-2.1.4.min.js') !!}

    @yield('head')

</head>

<body>

<div class="container">

    <div class="navbar navbar-default">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand" href="/">Dropzone + Laravel</a>
        </div>
        <div class="navbar-collapse collapse">
            <ul class="nav navbar-nav">
                <li><a href="/">Upload</a></li>
            </ul>
        </div>
    </div>

    <br><br>

@yield('content')

</div>
</body>

@yield('footer')
</html>
```

主要的布局文件非常简单，只是包含底部导航栏和多个balde的section。

```
@extends('layout')

@section('head')
    {!! HTML::style('/packages/dropzone/dropzone.css') !!}
@stop

@section('footer')
    {!! HTML::script('/packages/dropzone/dropzone.js') !!}
    {!! HTML::script('/assets/js/dropzone-config.js') !!}
@stop

@section('content')

    <div class="row">
        <div class="col-md-offset-1 col-md-10">
            <div class="jumbotron how-to-create" >

                <h3>Images <span id="photoCounter"></span></h3>
                <br />

                {!! Form::open(['url' => route('upload-post'), 'class' => 'dropzone', 'files'=>true, 'id'=>'real-dropzone']) !!}

                <div class="dz-message">

                </div>

                <div class="fallback">
                    <input name="file" type="file" multiple />
                </div>

                <div class="dropzone-previews" id="dropzonePreview"></div>

                <h4 style="text-align: center;color:#428bca;">Drop images in this area  <span class="glyphicon glyphicon-hand-down"></span></h4>

                {!! Form::close() !!}

            </div>
            <div class="jumbotron how-to-create">
                <ul>
                    <li>Images are uploaded as soon as you drop them</li>
                    <li>Maximum allowed size of image is 8MB</li>
                </ul>

            </div>
        </div>
    </div>

    <!-- Dropzone Preview Template -->
    <div id="preview-template" style="display: none;">

        <div class="dz-preview dz-file-preview">
            <div class="dz-image"><img data-dz-thumbnail=""></div>

            <div class="dz-details">
                <div class="dz-size"><span data-dz-size=""></span></div>
                <div class="dz-filename"><span data-dz-name=""></span></div>
            </div>
            <div class="dz-progress"><span class="dz-upload" data-dz-uploadprogress=""></span></div>
            <div class="dz-error-message"><span data-dz-errormessage=""></span></div>

            <div class="dz-success-mark">
                <svg width="54px" height="54px" viewBox="0 0 54 54" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:sketch="http://www.bohemiancoding.com/sketch/ns">
                    <!-- Generator: Sketch 3.2.1 (9971) - http://www.bohemiancoding.com/sketch -->
                    <title>Check</title>
                    <desc>Created with Sketch.</desc>
                    <defs></defs>
                    <g id="Page-1" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd" sketch:type="MSPage">
                        <path d="M23.5,31.8431458 L17.5852419,25.9283877 C16.0248253,24.3679711 13.4910294,24.366835 11.9289322,25.9289322 C10.3700136,27.4878508 10.3665912,30.0234455 11.9283877,31.5852419 L20.4147581,40.0716123 C20.5133999,40.1702541 20.6159315,40.2626649 20.7218615,40.3488435 C22.2835669,41.8725651 24.794234,41.8626202 26.3461564,40.3106978 L43.3106978,23.3461564 C44.8771021,21.7797521 44.8758057,19.2483887 43.3137085,17.6862915 C41.7547899,16.1273729 39.2176035,16.1255422 37.6538436,17.6893022 L23.5,31.8431458 Z M27,53 C41.3594035,53 53,41.3594035 53,27 C53,12.6405965 41.3594035,1 27,1 C12.6405965,1 1,12.6405965 1,27 C1,41.3594035 12.6405965,53 27,53 Z" id="Oval-2" stroke-opacity="0.198794158" stroke="#747474" fill-opacity="0.816519475" fill="#FFFFFF" sketch:type="MSShapeGroup"></path>
                    </g>
                </svg>
            </div>

            <div class="dz-error-mark">
                <svg width="54px" height="54px" viewBox="0 0 54 54" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:sketch="http://www.bohemiancoding.com/sketch/ns">
                    <!-- Generator: Sketch 3.2.1 (9971) - http://www.bohemiancoding.com/sketch -->
                    <title>error</title>
                    <desc>Created with Sketch.</desc>
                    <defs></defs>
                    <g id="Page-1" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd" sketch:type="MSPage">
                        <g id="Check-+-Oval-2" sketch:type="MSLayerGroup" stroke="#747474" stroke-opacity="0.198794158" fill="#FFFFFF" fill-opacity="0.816519475">
                            <path d="M32.6568542,29 L38.3106978,23.3461564 C39.8771021,21.7797521 39.8758057,19.2483887 38.3137085,17.6862915 C36.7547899,16.1273729 34.2176035,16.1255422 32.6538436,17.6893022 L27,23.3431458 L21.3461564,17.6893022 C19.7823965,16.1255422 17.2452101,16.1273729 15.6862915,17.6862915 C14.1241943,19.2483887 14.1228979,21.7797521 15.6893022,23.3461564 L21.3431458,29 L15.6893022,34.6538436 C14.1228979,36.2202479 14.1241943,38.7516113 15.6862915,40.3137085 C17.2452101,41.8726271 19.7823965,41.8744578 21.3461564,40.3106978 L27,34.6568542 L32.6538436,40.3106978 C34.2176035,41.8744578 36.7547899,41.8726271 38.3137085,40.3137085 C39.8758057,38.7516113 39.8771021,36.2202479 38.3106978,34.6538436 L32.6568542,29 Z M27,53 C41.3594035,53 53,41.3594035 53,27 C53,12.6405965 41.3594035,1 27,1 C12.6405965,1 1,12.6405965 1,27 C1,41.3594035 12.6405965,53 27,53 Z" id="Oval-2" sketch:type="MSShapeGroup"></path>
                        </g>
                    </g>
                </svg>
            </div>

        </div>
    </div>
    <!-- End Dropzone Preview Template -->

{!! Form::hidden('csrf-token', csrf_token(), ['id' => 'csrf-token']) !!}
@stop
```

在上传视图的 __头部部分__ 加入Dropzone的缺省样式，在 __底部部分__ 加入Dropzone缺省的js文件，和自己的Dropzone配置文件。

内容部分包含两个，第一个是上传表单部分，第二部分是隐藏的预览模板部分。在Dropzone下载了预览模板的代码。配置文件中的模板部分用来显示上传文件的预览。

因为需要使用token完成提交和删除路由，所以在内容的底部部分，加入隐藏的csrf字段。

## Dropzone的配置

Dropzone具有很多可用的配置选项。对于这个项目来说，使用Dropzone实现文件的批量顺序上传，就可以避免重复选择需要上传的文件。并行的上传数最大100个，单个文件的尺寸最大设置为8MB。同时上传成功的文件可以在模板html页面中显示预览。

每个预览的图片都可以通过点击下方的删除链接删除，这个操作会触发删除文件事件。这个事件会创建一个ajax发起一个删除请求，并且会返回200状态码，图片数也会相应的减少。

```
var photo_counter = 0;
Dropzone.options.realDropzone = {

    uploadMultiple: false,
    parallelUploads: 100,
    maxFilesize: 8,
    previewsContainer: '#dropzonePreview',
    previewTemplate: document.querySelector('#preview-template').innerHTML,
    addRemoveLinks: true,
    dictRemoveFile: 'Remove',
    dictFileTooBig: 'Image is bigger than 8MB',

    // The setting up of the dropzone
    init:function() {

        this.on("removedfile", function(file) {

            $.ajax({
                type: 'POST',
                url: 'upload/delete',
                data: {id: file.name, _token: $('#csrf-token').val()},
                dataType: 'html',
                success: function(data){
                    var rep = JSON.parse(data);
                    if(rep.code == 200)
                    {
                        photo_counter--;
                        $("#photoCounter").text( "(" + photo_counter + ")");
                    }

                }
            });

        } );
    },
    error: function(file, response) {
        if($.type(response) === "string")
            var message = response; //dropzone sends it's own error messages in string
        else
            var message = response.message;
        file.previewElement.classList.add("dz-error");
        _ref = file.previewElement.querySelectorAll("[data-dz-errormessage]");
        _results = [];
        for (_i = 0, _len = _ref.length; _i < _len; _i++) {
            node = _ref[_i];
            _results.push(node.textContent = message);
        }
        return _results;
    },
    success: function(file,done) {
        photo_counter++;
        $("#photoCounter").text( "(" + photo_counter + ")");
    }
}

```

在配置文件底部，当文件上传成功后，图片计数器会增加。

## 上传逻辑

根据功能逻辑划分不同的实现类，在这个项目中，实现了一个__ImageRepository__类，将其放到目录app/Logic/Image/ImageRepository.php中。

迄今为止这个类只有2个功能，主要是负责上传单个文件和删除指定文件。在__ImageController__类文件中实现了这个类的实例。

```
<?php

namespace App\Http\Controllers;

use App\Logic\Image\ImageRepository;
use Illuminate\Support\Facades\Input;

class ImageController extends Controller
{
    protected $image;

    public function __construct(ImageRepository $imageRepository)
    {
        $this->image = $imageRepository;
    }

    public function getUpload()
    {
        return view('pages.upload');
    }

    public function postUpload()
    {
        $photo = Input::all();
        $response = $this->image->upload($photo);
        return $response;

    }

    public function deleteUpload()
    {

        $filename = Input::get('id');

        if(!$filename)
        {
            return 0;
        }

        $response = $this->image->delete( $filename );

        return $response;
    }
}
```

因此这个控制器需要添加3个路由，一个用来显示上传表单，一个用来接收上传的文件，第三个用来发起删除请求。

```
Route::get('/', ['as' => 'upload', 'uses' => 'ImageController@getUpload']);
Route::post('upload', ['as' => 'upload-post', 'uses' =>'ImageController@postUpload']);
Route::post('upload/delete', ['as' => 'upload-remove', 'uses' =>'ImageController@deleteUpload']);

```

在类文件__ImageController__ 的 __postUpload__ 方法中将用户的输入传入  __ImageRepository__ 类中的 __upload__ 方法中。

这个方法

  * 首先会验证输入的合法性
  * 然后会为上传的文件生成服务器唯一的文件名
  * 最后将文件按照原始尺寸和icon的尺寸分别存入服务器不同的目录。

同时，他也可以在数据库中创建完整的实体，从而Laravel可以轻松的掌控上传的图片。

首先，创建migration文件，并执行migrations。

```
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateImages extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('images', function (Blueprint $table) {
            $table->increments('id');
            $table->text('original_name');
            $table->text('filename');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('images');
    }
}
```

注意到上传的文件会被保存为两种格式，原始尺寸和icon尺寸。使用 __Image Intervention package__ 修改图片尺寸和编码。

```
<?php

namespace App\Logic\Image;

use Illuminate\Support\Facades\Validator;
use Illuminate\Support\Facades\Response;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\File;
use Intervention\Image\ImageManager;
use App\Models\Image;


class ImageRepository
{
    public function upload( $form_data )
    {

        $validator = Validator::make($form_data, Image::$rules, Image::$messages);

        if ($validator->fails()) {

            return Response::json([
                'error' => true,
                'message' => $validator->messages()->first(),
                'code' => 400
            ], 400);

        }

        $photo = $form_data['file'];

        $originalName = $photo->getClientOriginalName();
        $extension = $photo->getClientOriginalExtension();
        $originalNameWithoutExt = substr($originalName, 0, strlen($originalName) - strlen($extension) - 1);

        $filename = $this->sanitize($originalNameWithoutExt);
        $allowed_filename = $this->createUniqueFilename( $filename, $extension );

        $uploadSuccess1 = $this->original( $photo, $allowed_filename );

        $uploadSuccess2 = $this->icon( $photo, $allowed_filename );

        if( !$uploadSuccess1 || !$uploadSuccess2 ) {

            return Response::json([
                'error' => true,
                'message' => 'Server error while uploading',
                'code' => 500
            ], 500);

        }

        $sessionImage = new Image;
        $sessionImage->filename      = $allowed_filename;
        $sessionImage->original_name = $originalName;
        $sessionImage->save();

        return Response::json([
            'error' => false,
            'code'  => 200
        ], 200);

    }

    public function createUniqueFilename( $filename, $extension )
    {
        $full_size_dir = Config::get('images.full_size');
        $full_image_path = $full_size_dir . $filename . '.' . $extension;

        if ( File::exists( $full_image_path ) )
        {
            // Generate token for image
            $imageToken = substr(sha1(mt_rand()), 0, 5);
            return $filename . '-' . $imageToken . '.' . $extension;
        }

        return $filename . '.' . $extension;
    }

    /**
     * Optimize Original Image
     */
    public function original( $photo, $filename )
    {
        $manager = new ImageManager();
        $image = $manager->make( $photo )->save(Config::get('images.full_size') . $filename );

        return $image;
    }

    /**
     * Create Icon From Original
     */
    public function icon( $photo, $filename )
    {
        $manager = new ImageManager();
        $image = $manager->make( $photo )->resize(200, null, function ($constraint) {
            $constraint->aspectRatio();
            })
            ->save( Config::get('images.icon_size')  . $filename );

        return $image;
    }

    /**
     * Delete Image From Session folder, based on original filename
     */
    public function delete( $originalFilename)
    {

        $full_size_dir = Config::get('images.full_size');
        $icon_size_dir = Config::get('images.icon_size');

        $sessionImage = Image::where('original_name', 'like', $originalFilename)->first();


        if(empty($sessionImage))
        {
            return Response::json([
                'error' => true,
                'code'  => 400
            ], 400);

        }

        $full_path1 = $full_size_dir . $sessionImage->filename;
        $full_path2 = $icon_size_dir . $sessionImage->filename;

        if ( File::exists( $full_path1 ) )
        {
            File::delete( $full_path1 );
        }

        if ( File::exists( $full_path2 ) )
        {
            File::delete( $full_path2 );
        }

        if( !empty($sessionImage))
        {
            $sessionImage->delete();
        }

        return Response::json([
            'error' => false,
            'code'  => 200
        ], 200);
    }

    function sanitize($string, $force_lowercase = true, $anal = false)
    {
        $strip = array("~", "`", "!", "@", "#", "$", "%", "^", "&", "*", "(", ")", "_", "=", "+", "[", "{", "]",
            "}", "\\", "|", ";", ":", "\"", "'", "&#8216;", "&#8217;", "&#8220;", "&#8221;", "&#8211;", "&#8212;",
            "â€”", "â€“", ",", "<", ".", ">", "/", "?");
        $clean = trim(str_replace($strip, "", strip_tags($string)));
        $clean = preg_replace('/\s+/', "-", $clean);
        $clean = ($anal) ? preg_replace("/[^a-zA-Z0-9]/", "", $clean) : $clean ;

        return ($force_lowercase) ?
            (function_exists('mb_strtolower')) ?
                mb_strtolower($clean, 'UTF-8') :
                strtolower($clean) :
            $clean;
    }
}
```

上传的方法中还会引用Image模型中验证规则验证图片类型，只有符合条件的图片可以上传。

然后，通过方法 __createUniqueFilename__ 检查下图片原始的文件名称在服务器上是否唯一。如果存在同名的文件，这个方法会在文件名后追加随机字符串。这是最简单的解决办法，当然了，你可以通过其他方法实现。

当获取到唯一的文件名称后，将图片和这个文件名传给方法original() ，这个方法负责保存全尺寸的图片，使用icon()方法保存小尺寸图片。

当这两个尺寸的图片被保存后，系统会在images表中插入新的记录。

图片上传的目录路径被保存在图片配置文件中。同时使用了自己的sanitize()方法，你也可以使用Laravel内部自建的sanitize().

为了项目能正常的工作，需要拷贝.env.example 到.env文件，并且修改其中的值。上传目录需要以"/"结束。

```
<?php

return [
    'full_size'   => env('UPLOAD_FULL_SIZE'),
    'icon_size'   => env('UPLOAD_ICON_SIZE'),
];
```

删除方法会接受文件名，并且检查这个图片是否存在数据库和对应的两个目录下。如果存在则将其删除。p

## 来源
  * https://tuts.codingo.me/laravel-5-1-and-dropzone-js-auto-image-uploads-with-removal-links#
