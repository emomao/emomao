<html>
<head>
    <meta charset="UTF-8">
    <title>短视频解析</title>
    <meta name="renderer" content="webkit">
    <meta name="referrer" content="never">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
    <link href="https://cdn.bootcdn.net/ajax/libs/layui/2.7.6/css/layui.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/layui/2.7.6/layui.min.js"></script>
</head>
<style>
.body {max-width: 500px;margin: auto;}
.main {padding:6px 6px;margin:auto;background-color: white;}
.tt {color: #1aa700;font-size: 1.2rem;font-weight: 700;padding: 8px;}
.center {text-align: center;}
</style>
<body>
    <div class="main">
        <blockquote class="layui-elem-quote tt">短视频解析去水印</blockquote>
        <div class="layui-row">
            <div class="layui-col-md12">
                <div class="layui-card-body" id="player" style="display: none;">
                    <div class="layui-form-item" style="margin: 0 10px 10px 10px;">
                        <video name="video" id="video" width="100%" controls autoplay loop></video>
                    </div>
                </div>
            </div>
            <div class="layui-col-md12">
                <div class="layui-card-body">
                    <form class="layui-form layui-form-pane" action="">
                        <div class="layui-form-item layui-form-text">
                            <label class="layui-form-label">输入分享链接</label>
                            <div class="layui-input-block">
                            <textarea name="link" id="link" placeholder="请输入内容" class="layui-textarea"> https://v.douyin.com/iLCdDjht/ </textarea>
                            </div>
                        </div>
                        <input type="text" name="downloadurl" style="display: none;">
                        <input type="text" name="filename" style="display: none;">
                        <div class="layui-form-item center">
                        <button class="layui-btn" lay-submit="" lay-filter="Submit">提交</button>
                        <button class="layui-btn" lay-submit="" lay-filter="Remove">再来一个</button>
                        <button class="layui-btn" lay-submit="" id="download" style="display: none;" lay-filter="Download">下载</button>
                        </div>
                    </form>
                </div>
                <div id="Result" style="display: none;">
                    <div class="layui-card-header">解析结果</div>
                    <div class="layui-card-body">
                        <div class="layui-field-box">
                            <div style="margin-top: 0px;">
                            <p><span class="layui-badge">uid</span> <span id="uid"></span></p>
                            <p><span class="layui-badge">author</span> <span id="author"></span></p>
                            <p><span class="layui-badge">create_time</span> <span id="create_time"></span></p>
                            <p><span class="layui-badge">desc</span> <span id="desc"></span></p>
                            <p><span class="layui-badge">video_id</span> <span id="video_id"></span></p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>
<script>
    layui.use(['form'], function(){
        var form = layui.form
        ,$ = layui.jquery
        ,layer = layui.layer;
 
        form.on('submit(Submit)', function(data){
            var link = data.field.link;
            if (link.length === 0) {
                layer.alert('请输入您要解析的内容！', { title: '提示' })
                return false;
            }
 
            var i = link.lastIndexOf("https://");
            i = i === -1 ? link.lastIndexOf("http://") : i;
            var url = link.substr(i);
            var index = layer.load(0, {shade: false});
            $.ajax({
                type: 'GET',
                url: 'https://api.qoc.cc/api/video?url=' + url,
                success: function(s) {
                    if (s.code === 200) {
                        var filename = s.data.title
                        var videourl = s.data.url;
                        $('#author').html(s.data.author);
                        $('#uid').html(s.data.uid);
                        $('#create_time').html(s.data.time);
                        $('#desc').html(s.data.title);
                        $('#video_id').html(s.data.like);
                        $('#title').html(filename);
                        $('#download').show();
                        $('#Result').show();
                        $('#vice').show();
                        downloadBlobFile('get', videourl).onreadystatechange = res=>{
                            if (res.currentTarget.readyState == 4 && res.currentTarget.status == 200) {
                                const url = window.URL.createObjectURL(res.currentTarget.response);
                                $('#video').attr('src',url);
                                $('#player').show();
                                $("input[name=downloadurl]").val(url);
                            }
                        }
                        $("input[name=filename]").val(filename);
                        document.title = filename;
                    } else {
                        layer.msg(s.message);
                    }
                    layer.close(index);
                }
            });
            return false;
        });
 
        form.on('submit(Remove)', function(data){
            $("#Result").hide();
            $("#link").val('');
            $('#video').attr('src','');
            $('#download').hide();
            $('#player').hide();
            $('#vice').hide();
            return false;
        });
 
        form.on('submit(Download)', function(data){
            downloadBlobFile('get',data.field.downloadurl).onreadystatechange = res=>{
                if(res.currentTarget.readyState == 4 &&  res.currentTarget.status==200){
                    const url = window.URL.createObjectURL(res.currentTarget.response);
                    let a = document.createElement('a');
                    a.href=url;
                    a.download = data.field.filename;
                    a.click();
                }
            }
            return false;
        });
 
        function downloadBlobFile(_method,_url){
            const request = new XMLHttpRequest();
            request.open(_method,_url);
            request.send();
            request.responseType = 'blob';
            return request;
        }
        function isClipboardAPIEnabled() {
            return !!(navigator.clipboard && navigator.clipboard.readText);
        }
        function addClipboardEventListener() {
            var pasteButton = document.getElementById('paste-button');
            pasteButton.addEventListener('click', async function() {
                try {
                    var text = await navigator.clipboard.readText();
                    $('#link').val(text);
                } catch (err) {
                    console.error('An error occurred while reading clipboard contents:', err);
                }
            });
        }
        if (!isClipboardAPIEnabled()) {
            document.getElementById('paste-button').style.display = 'block';
            addClipboardEventListener();
        }
    });
</script>
</html>
