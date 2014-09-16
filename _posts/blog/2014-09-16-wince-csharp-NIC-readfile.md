---
layout: post
title: wince下用C#读写网卡
description: 总结wince7下c#读写网卡遇到的问题
category: blog
---

**环境**
- wince7
- arm架构cpu
- .net compact framwork 3.5

**注意事项**

1. DeviceIoControl中用到的*IOCTL_NDISPROT_OPEN_DEVICE *、*IOCTL_NDISUIO_SET_ETHER_TYPE *
ce下与pc下值是不一样的
2. 若要接收数据包，必须设置以太网类型


- - -

**代码片**

```c#
using System;
using System.Linq;
using System.Collections.Generic;
using System.Text;
using System.Threading;
using System.Runtime.InteropServices;
using System.IO;
//using System.Net.NetworkInformation;
using System.Diagnostics;
//using Microsoft.Win32.SafeHandles;

namespace HMI
{
    public class CommDevice
    {
        #region IMPORTS
        [DllImport("coredll", SetLastError = true)]
        public static extern IntPtr CreateFile(
            string _lpFileName,				// filename to open
            uint _dwDesiredAccess,			// access permissions for the file
            uint _dwShareMode,				// sharing or locked
            uint _lpSecurityAttributes,		// security attributes
            uint _dwCreationDisposition,	// file creation method (new, existing)
            uint _dwFlagsAndAttributes,		// other flags and sttribs
            uint _hTemplateFile);			// template file for creating
        //not good
        //[DllImport("coredll", SetLastError = true)]
        //public static extern bool DeviceIoControl(
        //    //SafeFileHandle hDevice,
        //    IntPtr hDevice,
        //    uint dwIoControlCode,
        //    string lpInBuffer,
        //    uint nInBufferSize,
        //    IntPtr lpOutBuffer,
        //    uint nOutBufferSize,
        //    out uint lpbytesReturned,
        //    IntPtr lpOverlapped);
        [DllImport("coredll", SetLastError = true)]
        public static extern bool DeviceIoControl(
            IntPtr hDevice,
            uint dwIoControlCode,
            IntPtr lpInBuffer,
            uint nInBufferSize,
            IntPtr lpOutBuffer,
            uint nOutBufferSize,
            out uint lpbytesReturned,
            IntPtr lpOverlapped);
        [System.Runtime.InteropServices.DllImport("coredll", SetLastError = true)]
        static extern bool WriteFile(
            IntPtr File,                // handle to file
            byte[] Buffer,                  // data buffer
            int NumberOfBytesToWrite,       // number of bytes to write
            out int NumberOfBytesWritten,  // number of bytes write
            int Overlapped                  // overlapped buffer
        );
        [DllImport("coredll", SetLastError = true)]
        static extern bool ReadFile(
            IntPtr File,
            byte[] Buffer,               // data buffer
            int NumberOfBytesToRead,     // number of bytes to read
            out int NumberOfBytesRead,  // number of bytes read
            int Overlapped               // overlapped buffer
        );
        [DllImport("kernel32.dll")]
        //public static extern int CloseHandle(SafeFileHandle hObject);
        public static extern int CloseHandle(IntPtr hObject);
        #endregion

        #region CONSTANTS
        public const uint GENERIC_READ = 0x80000000;
        public const uint GENERIC_WRITE = 0x40000000;
        public const uint OPEN_EXISTING = 0x00000003;
        public const uint FILE_ATTRIBUTE_NORMAL = 0x00000080;
        public const uint FILE_FLAG_OVERLAPPED = 0x40000000;
        public const uint IOCTL_NDISPROT_BIND_WAIT = 0x0012c810;
        public const uint IOCTL_NDISPROT_OPEN_DEVICE = 0x00120800;
        public const uint IOCTL_NDISUIO_SET_ETHER_TYPE = 0x00120808;
        //public const uint IOCTL_NDISPROT_OPEN_DEVICE = 0x0012c800;
        #endregion

        #region ATTRIBUTES
        private string m_sNdisProtDriver = "UIO1:";
        private string adapterID = "CPSW3G2";
        #endregion
        private string NetID;
        //private SafeFileHandle Handle = null;
        private IntPtr Handle = IntPtr.Zero;
        private CommDevice(string mNetID)
        {
            NetID = mNetID;// "\\DEVICE\\{94607816-E437-404B-A015-C81DA8AA0D85}";
        }
        private static CommDevice device = null;
        public static CommDevice GetCommDevice(string mNetID)
        {
            if (device == null) device = new CommDevice(mNetID);
            return device;
        }
        private void OpenHandle()
        {
            Handle = CreateFile(m_sNdisProtDriver,
                GENERIC_READ | GENERIC_WRITE,
                3,
                0,
                OPEN_EXISTING,
                FILE_ATTRIBUTE_NORMAL | FILE_FLAG_OVERLAPPED,
                //0x00);
                0xFFFFFFFF);

            if ((int)Handle <= 0)
            {
                throw new Exception("error open handle");
            }
            //uint BytesReturned;
            //if (!DeviceIoControl(Handle, IOCTL_NDISPROT_BIND_WAIT,
            //    IntPtr.Zero, 0, IntPtr.Zero, 0, out BytesReturned, IntPtr.Zero))
            //{
            //    throw new Exception("IOCTL_NDISIO_BIND_WAIT failed");
            //}
        }
        private void OpenNdisDevice()
        {
            uint BytesReturned;
            bool result = DeviceIoControl(Handle,
                     IOCTL_NDISPROT_OPEN_DEVICE,
                     Marshal.StringToBSTR(NetID),
                     (uint)NetID.Length * 2,
                     IntPtr.Zero,
                     0,
                     out BytesReturned,
                     IntPtr.Zero);
            ushort type = 0x0008;
            IntPtr ptype = Marshal.AllocHGlobal(Marshal.SizeOf(type));
            Marshal.StructureToPtr(type, ptype, false);
            bool result2 = DeviceIoControl(Handle,
                IOCTL_NDISUIO_SET_ETHER_TYPE,
                   ptype,
                    2,
                    IntPtr.Zero,
                    0,
                    out BytesReturned,
                    IntPtr.Zero);
            if (!(result && result2))
            {
                throw new Exception("IOCTL_NDISPROT_OPEN_DEVICE failed" + Marshal.GetLastWin32Error());
            }
            // 打开设备
            //             if (!DeviceIoControl(Handle, IOCTL_NDISPROT_OPEN_DEVICE,
            //                 NetID.ToCharArray(), (uint)NetID.Length * 2,
            //                 IntPtr.Zero, 0, out BytesReturned, IntPtr.Zero))
            //             {
            //                 throw new Exception("IOCTL_NDISPROT_OPEN_DEVICE failed" + Marshal.GetLastWin32Error());
            //             }

            // 打开设备
            //             if (!DeviceIoControl(Handle, IOCTL_NDISPROT_OPEN_DEVICE,
            //                 Marshal.StringToHGlobalUni(NetID), (uint)NetID.Length * 2,
            //                 IntPtr.Zero, 0, out BytesReturned, IntPtr.Zero))
            //             {
            //                 throw new Exception("IOCTL_NDISPROT_OPEN_DEVICE failed" + Marshal.GetLastWin32Error());
            //             }
        }

        public void openDevice()
        {
            OpenHandle();
            OpenNdisDevice();
        }

        int readCount;
        public int receive(byte[] buffer, int count)
        {
            int writecnt = 0;
            byte[] buff = new byte[] {
                0x6c, 0xf0, 0x49, 0x72, 0xaa, 0x2b,
                0xe0, 0xc7, 0x9d, 0xaa, 0x97, 0xf7,
                0x83, 0x82, 
                0x45, 0x00, 0x00, 0x30, 0x17, 0x6b, 0x40, 0x00, 
                0x80, 0x06, 0x61, 0xf7, 0xc0, 0xa8, 0x00, 0x0b, 0xc0,
                0xa8, 0x00, 0x0a, 0x16, 0x17, 0x37, 0x19, 0x41, 0x21,
                0xc8, 0x83, 0xdc, 0x19, 0xc7, 0xd9, 0x50, 0x18, 0x00, 
                0x1f, 0x32, 0x77, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 
                0x00, 0x00, 0x00, 0x00};
            //WriteFile(Handle, buff, buff.Length, out writecnt, 0);
            //byte[] bufferr=new byte[2048];
            //ReadFile(Handle, bufferr, bufferr.Length, out readCount, 0);
            ReadFile(Handle, buffer, count, out readCount, 0);
            return readCount;
        }
        public void stop()
        {
            if (Handle != null)
            {
                CloseHandle(Handle);
            }
        }
    }
}


```

[zhiying678]:    http://blog.houmingjiang.cn  "zhiying678"
