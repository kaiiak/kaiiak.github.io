title: C#利用VLC播放网络串流
date: 2015-11-19 21:08:23
tags: [vlc,CSharp]
---
[VLC的官方网站](http://www.videolan.org)，VLC的源代码是用C语言实现的。我们这里用`C#`的封装——nVLC。
[nVLC](http://www.codeproject.com/Articles/109639/nVLC)。

## 开始
在运行本程序之前，请先安装VLC，或者拷贝`libvlc.dll`、`libvlccode.dll`和`plugins目录`到你的

```c#
using System;
using System.Windows.Forms;
using Declarations;
using Declarations.Enums;
using Declarations.Media;
using Declarations.Players;
using Implementation;

namespace nVLC_Demo_MemoryInputOutput
{
    public partial class Form1 : Form
    {
        IMediaPlayerFactory m_factory;
        IVideoPlayer m_sourcePlayer;
        IVideoPlayer m_renderPlayer;
        IMemoryInputMedia m_inputMedia;
        const long MicroSecondsInSecomd = 1000 * 1000;
        long MicroSecondsBetweenFrame;
        long frameCounter;
        FrameData data = new FrameData() { DTS = -1 };
        const int DefaultFps = 24;
        Timer timer = new Timer();

        public Form1()
        {
            InitializeComponent();
            timer.Tick += new EventHandler(timer_Tick);
            timer.Interval = 1000;
        }

        void timer_Tick(object sender, EventArgs e)
        {
            this.Text = m_inputMedia.PendingFramesCount.ToString();
        }

        protected override void OnLoad(EventArgs e)
        {
            base.OnLoad(e);

            m_factory = new MediaPlayerFactory(true);
            m_sourcePlayer = m_factory.CreatePlayer<IVideoPlayer>();
            m_sourcePlayer.Events.PlayerPlaying += new EventHandler(Events_PlayerPlaying);
            m_sourcePlayer.Mute = true;
            m_renderPlayer = m_factory.CreatePlayer<IVideoPlayer>();
            m_renderPlayer.WindowHandle = panel1.Handle;
            m_inputMedia = m_factory.CreateMedia<IMemoryInputMedia>(MediaStrings.IMEM);
            SetupOutput(m_sourcePlayer.CustomRendererEx);
        }

        void Events_PlayerPlaying(object sender, EventArgs e)
        {
            MicroSecondsBetweenFrame = (long)(MicroSecondsInSecomd / (m_sourcePlayer.FPS != 0 ? m_sourcePlayer.FPS : DefaultFps));
        }

        private void SetupOutput(IMemoryRendererEx iMemoryRenderer)
        {
            iMemoryRenderer.SetFormatSetupCallback(OnSetupCallback);
            iMemoryRenderer.SetExceptionHandler(OnErrorCallback);
            iMemoryRenderer.SetCallback(OnNewFrameCallback);
        }

        private BitmapFormat OnSetupCallback(BitmapFormat format)
        {
            SetupInput(format);
            return new BitmapFormat(format.Width, format.Height, ChromaType.RV24);
        }

        private void OnErrorCallback(Exception error)
        {
            MessageBox.Show(error.Message);
        }

        private void OnNewFrameCallback(PlanarFrame frame)
        {          
            data.Data = frame.Planes[0];
            data.DataSize = frame.Lenghts[0];
            data.PTS = frameCounter++ * MicroSecondsBetweenFrame;
            m_inputMedia.AddFrame(data);

            if (/*m_inputMedia.PendingFramesCount == 10 && */!m_renderPlayer.IsPlaying)
            {
                m_renderPlayer.Play();
            }
        }

        private void SetupInput(BitmapFormat format)
        {
            var streamInfo = new StreamInfo();
            streamInfo.Category = StreamCategory.Video;
            streamInfo.Codec = VideoCodecs.BGR24;
            streamInfo.Width = format.Width;
            streamInfo.Height = format.Height;
            streamInfo.Size = format.ImageSize;

            m_inputMedia.Initialize(streamInfo);
            m_inputMedia.SetExceptionHandler(OnErrorCallback);
            m_renderPlayer.Open(m_inputMedia);           
        }

        private void OpenSourceMedia(string path)
        {
            //IMediaFromFile media = m_factory.CreateMedia<IMediaFromFile>(path);
            IMedia media = m_factory.CreateMedia<IMedia>(path);
            m_sourcePlayer.Open(media);
            m_sourcePlayer.Play();
            timer.Start();
            
        }

        private void button1_Click(object sender, EventArgs e)
        {
            OpenSourceMedia("http://192.168.2.108:8090");
        }
    }
}
```