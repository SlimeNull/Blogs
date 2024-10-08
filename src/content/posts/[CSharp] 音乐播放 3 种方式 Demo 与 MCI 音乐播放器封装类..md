---
title: '[C#] 音乐播放 3 种方式 Demo 与 MCI 音乐播放器封装类.'
slug: '20210211104239'
published: 2021-02-11T10:42:39
tags:
  - dotnet
  - csharp
  - win32
  - winapi
  - winform
category: '.NET'
description: '淦!首先代码是不完整的, 我还是没能成功设置音量, 除此之外, 别的倒是挺顺利的, 当前实现的功能如下:播放 暂停 还原 停止 获取进度 获取长度 设置进度 获取播放状态内容:本项目的仓库在: SlimeNull/AudioTest, 在这里下载哦, 别忘了还原 nuget 包, 因为用了 NAudio 库.> 音乐播放 Demo:先放图吧:首先左上角是打开文件的, 然后 PlayType 是播放方式, 我现在让它支持 3 种, 一种是SoundPlayer, 然后是 M'
---

## 淦!

首先代码是不完整的, 我还是没能成功设置音量, 除此之外, 别的倒是挺顺利的, 当前实现的功能如下:

> 播放 暂停 还原 停止 获取进度 获取长度 设置进度 获取播放状态

## 内容:

- 本项目的仓库在: <a id="download_link" href="https://github.com/SlimeNull/AudioTest">SlimeNull/AudioTest</a>, 在这里下载哦, 别忘了还原 nuget 包, 因为用了 NAudio 库.


#### > 音乐播放 Demo:

- 先放图吧:

![](/images/20210211102257257.png)

  > 首先左上角是打开文件的, 然后 PlayType 是播放方式, 我现在让它支持 3 种, 一种是SoundPlayer, 然后是 MCI, 然后是 NAudio 库.
  > &nbsp;
  > 实现了播放, 暂停, 停止, 移除, 进度调整到开头, 进度调整(就那个track bar), 同步播放(如果点了这个, UI线程会阻塞的哦), 获取音乐长度, 获取播放状态, 获取播放位置, 重复播放.
  > &nbsp;
  > 不同的功能对不同的播放种类支持性不同, 所以, 如果你按下了按钮, 没反应, 也没报错, 那就是它不支持这个功能(也就是代码里没有写)'

   下载代码, 请看前面的链接.[↑跳转↑](#download_link)
   

#### > MCI 播放器封装:

- 上边的窗体程序就体现了封装类所实现的所有功能喔, 所以试试吧, 你也可以直接粘贴下面的代码, 但是cs文件的名称, 如果可以的话, 我希望你命名为 "Null.MciPlayer" <sub>(没错就是我的名字</sub>
    ```csharp
    using System;
    using System.IO;
    using System.Runtime.InteropServices;
    
    namespace Null.MciPlayer
    {
        public class MciPlayer : IDisposable
        {
            [DllImport("kernel32.dll", EntryPoint = "GetShortPathNameW", CharSet = CharSet.Unicode)]
            extern static short GetShortPath(string longPath, string buffer, int bufferSize);
            [DllImport("winmm.dll", EntryPoint = "mciSendString", CharSet = CharSet.Unicode)]
            extern static int MciSendString(string command, string buffer, int bufferSize, IntPtr callback);
    
            bool TryGetShortPath(string longPath, out string shortPath)
            {
                shortPath = null;
                short reqLen = GetShortPath(longPath, null, 0);   // 指定 null与0, 则返回需要的长度
                if (reqLen == 0)
                    return false;
    
                shortPath = new string('\0', reqLen);   // 声明缓冲
    
                short rstLen = GetShortPath(longPath, shortPath, reqLen);   // 转换
                if (rstLen == 0 || rstLen == reqLen)
                    return false;
    
                shortPath = shortPath.TrimEnd('\0');
                return true;
            }
    
            private string longpath;
            private string shortName;
            private string aliasName;
            public MciPlayer() { }
            public MciPlayer(string path)
            {
                if (!File.Exists(path))
                    throw new FileNotFoundException("File not exist.", path);
    
                longpath = path;
            }
            void MciSendStringWithCheck(string command, string buffer, int bufferSize, IntPtr callback)
            {
                int err = MciSendString(command, buffer, bufferSize, callback);
                if (err != 0)
                    throw new MciException(err);
            }
            string StatusInfo(string info)
            {
                string buffer = new string('\0', 32);
                MciSendStringWithCheck($"status {aliasName} {info}", buffer, 32, IntPtr.Zero);
    
                return buffer.TrimEnd('\0');
            }
    
    
    
            public string DevicePath { get => longpath; }
            public string DeviceShortPath { get => shortName; }
            public string AliasName { get => aliasName; }
            public bool SetDevicePath(string longpath)
            {
                if (aliasName != null)
                    return false;
    
                this.longpath = longpath;
                return true;
            }
            public void Open()
            {
                if (!TryGetShortPath(longpath, out shortName))
                    throw new Exception("Get short path faield when initializing.");
    
                aliasName = $"nmci{DateTime.Now.Ticks}";
                MciSendStringWithCheck($"open {shortName} alias {aliasName}", null, 0, IntPtr.Zero);
            }
            public void Close()
            {
                MciSendStringWithCheck($"close {aliasName}", null, 0, IntPtr.Zero);
    
                aliasName = null;
            }
            public void Play()
            {
                MciSendStringWithCheck($"play {aliasName}", null, 0, IntPtr.Zero);
            }
            public void Resume()
            {
                MciSendStringWithCheck($"resume {aliasName}", null, 0, IntPtr.Zero);
            }
            public void Pause()
            {
                MciSendStringWithCheck($"pause {aliasName}", null, 0, IntPtr.Zero);
            }
            public void Stop()
            {
                MciSendStringWithCheck($"stop {aliasName}", null, 0, IntPtr.Zero);
            }
            public int GetPosition()
            {
                return int.Parse(StatusInfo("position"));
            }
            public int GetLength()
            {
                return int.Parse(StatusInfo("length"));
            }
            public PlaybackState GetState()
            {
                switch (StatusInfo("mode").ToLower())
                {
                    case "playing":
                        return PlaybackState.Playing;
                    case "paused":
                        return PlaybackState.Paused;
                    case "stopped":
                        return PlaybackState.Stopped;
                    default:
                        return PlaybackState.Invalid;
                }
            }
            public void PlayWait()
            {
                MciSendStringWithCheck($"play {aliasName} wait", null, 0, IntPtr.Zero);
            }
            public void PlayRepeat()
            {
                MciSendStringWithCheck($"play {aliasName} repeat", null, 0, IntPtr.Zero);
            }
            public void Seek(int position)
            {
                MciSendStringWithCheck($"seek {aliasName} to {position}", null, 0, IntPtr.Zero);
            }
            public void SeekToStart()
            {
                MciSendStringWithCheck($"seek {aliasName} to start", null, 0, IntPtr.Zero);
            }
            public void SetSeekMode(bool fExact)
            {
                MciSendStringWithCheck($"set {aliasName} seek exactly {(fExact ? "on" : "off")}", null, 0, IntPtr.Zero);
            }
            public void Dispose()
            {
                if (aliasName != null)
                    Close();
            }
    
        }
        public enum PlaybackState
        {
            Stopped,
            Playing,
            Paused,
    
            Invalid = -1,
        }
        public class MciException : Exception
        {
            private MciError err;
    
            public int ErrorId { get => (int)err; }
            public string ErrorName { get => err.ToString(); }
            public override string Message { get => ErrorName; }
            public MciException() { }
            public MciException(int errorId)
            {
                if (Enum.IsDefined(typeof(MciError), errorId))
                    err = (MciError)errorId;
                else
                    throw new ArgumentOutOfRangeException("不是正确的错误ID");
            }
            public MciException(string errorName)
            {
                if (!Enum.TryParse<MciError>(errorName, out err))
                    throw new ArgumentOutOfRangeException("不是正确的错误ID");
            }
            enum MciError
            {
                MCIERR_NO_ERROR = 0,
    
                MCIERR_INVALID_DEVICE_ID = 257,
                MCIERR_UNRECOGNIZED_KEYWORD = 259,
                MCIERR_UNRECOGNIZED_COMMAND = 261,
                MCIERR_HARDWARE = 262,
                MCIERR_INVALID_DEVICE_NAME = 263,
                MCIERR_OUT_OF_MEMORY = 264,
                MCIERR_DEVICE_OPEN = 265,
                MCIERR_CANNOT_LOAD_DRIVER = 266,
                MCIERR_MISSING_COMMAND_STRING = 267,
                MCIERR_PARAM_OVERFLOW = 268,
                MCIERR_MISSING_STRING_ARGUMENT = 269,
                MCIERR_BAD_INTEGER = 270,
                MCIERR_PARSER_INTERNAL = 271,
                MCIERR_DRIVER_INTERNAL = 272,
                MCIERR_MISSING_PARAMETER = 273,
                MCIERR_UNSUPPORTED_FUNCTION = 274,
                MCIERR_FILE_NOT_FOUND = 275,
                MCIERR_DEVICE_NOT_READY = 276,
                MCIERR_INTERNAL = 277,
                MCIERR_DRIVER = 278,
                MCIERR_CANNOT_USE_ALL = 279,
                MCIERR_MULTIPLE = 280,
                MCIERR_EXTENSION_NOT_FOUND = 281,
                MCIERR_OUTOFRANGE = 282,
                MCIERR_FLAGS_NOT_COMPATIBLE = 284,
                MCIERR_FILE_NOT_SAVED = 286,
                MCIERR_DEVICE_TYPE_REQUIRED = 287,
                MCIERR_DEVICE_LOCKED = 288,
                MCIERR_DUPLICATE_ALIAS = 289,
                MCIERR_BAD_CONSTANT = 290,
                MCIERR_MUST_USE_SHAREABLE = 291,
                MCIERR_MISSING_DEVICE_NAME = 292,
                MCIERR_BAD_TIME_FORMAT = 293,
                MCIERR_NO_CLOSING_QUOTE = 294,
                MCIERR_DUPLICATE_FLAGS = 295,
                MCIERR_INVALID_FILE = 296,
                MCIERR_NULL_PARAMETER_BLOCK = 297,
                MCIERR_UNNAMED_RESOURCE = 298,
                MCIERR_NEW_REQUIRES_ALIAS = 299,
                MCIERR_NOTIFY_ON_AUTO_OPEN = 300,
                MCIERR_NO_ELEMENT_ALLOWED = 301,
                MCIERR_NONAPPLICABLE_FUNCTION = 302,
                MCIERR_ILLEGAL_FOR_AUTO_OPEN = 303,
                MCIERR_FILENAME_REQUIRED = 304,
                MCIERR_EXTRA_CHARACTERS = 305,
                MCIERR_DEVICE_NOT_INSTALLED = 306,
                MCIERR_GET_CD = 307,
                MCIERR_SET_CD = 308,
                MCIERR_SET_DRIVE = 309,
                MCIERR_DEVICE_LENGTH = 310,
                MCIERR_DEVICE_ORD_LENGTH = 311,
                MCIERR_NO_INTEGER = 312,
    
                MCIERR_WAVE_OUTPUTSINUSE = 320,
                MCIERR_WAVE_SETOUTPUTINUSE = 321,
                MCIERR_WAVE_INPUTSINUSE = 322,
                MCIERR_WAVE_SETINPUTINUSE = 323,
                MCIERR_WAVE_OUTPUTUNSPECIFIED = 324,
                MCIERR_WAVE_INPUTUNSPECIFIED = 325,
                MCIERR_WAVE_OUTPUTSUNSUITABLE = 326,
                MCIERR_WAVE_SETOUTPUTUNSUITABLE = 327,
                MCIERR_WAVE_INPUTSUNSUITABLE = 328,
                MCIERR_WAVE_SETINPUTUNSUITABLE = 329,
    
                MCIERR_SEQ_DIV_INCOMPATIBLE = 336,
                MCIERR_SEQ_PORT_INUSE = 337,
                MCIERR_SEQ_PORT_NONEXISTENT = 338,
                MCIERR_SEQ_PORT_MAPNODEVICE = 339,
                MCIERR_SEQ_PORT_MISCERROR = 340,
                MCIERR_SEQ_TIMER = 341,
                MCIERR_SEQ_PORTUNSPECIFIED = 342,
                MCIERR_SEQ_NOMIDIPRESENT = 343,
    
                MCIERR_NO_WINDOW = 346,
                MCIERR_CREATEWINDOW = 347,
                MCIERR_FILE_READ = 348,
                MCIERR_FILE_WRITE = 349,
                MCIERR_NO_IDENTITY = 350,
            }
        }
    }
    ```


<br/><br/><br/><br/><br/>
如果有什么建议或者这个类有可改进的地方, 欢迎留言哦\~  n(\*≧▽≦\*)n

