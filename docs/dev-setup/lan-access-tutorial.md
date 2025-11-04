# How to Access Local Servers from Other Devices in the Same LAN
# 如何从同一局域网内的其他设备访问本地服务器

> **Notice:** This is an AI-generated document and is for reference only.
> **注意:** 本文档由AI生成，仅供参考。

This guide explains how to configure your Django and Vue.js development servers to be accessible from other devices on the same local area network (LAN).
本指南介绍了如何配置Django和Vue.js开发服务器，以便在同一局域网（LAN）内的其他设备上访问。

## 1. Find Your Local IP Address
## 1. 查找您的本地IP地址

First, you need to find the local IP address of the computer running the development servers.
首先，您需要找到运行开发服务器的计算机的本地IP地址。

- **On Windows:** Open Command Prompt or PowerShell and run `ipconfig`. Look for the "IPv4 Address" in the output under your active network adapter (e.g., "Wireless LAN adapter Wi-Fi" or "Ethernet adapter Ethernet").
- **在Windows上:** 打开命令提示符或PowerShell并运行 `ipconfig`。 在活动网络适配器（例如，“无线局域网适配器 Wi-Fi”或“以太网适配器以太网”）下查找输出中的“IPv4 地址”。

- **On macOS:** Open Terminal and run `ifconfig | grep "inet "`. Look for the IP address that is not `127.0.0.1`.
- **在macOS上:** 打开终端并运行 `ifconfig | grep "inet "`。 查找不是 `127.0.0.1` 的IP地址。

- **On Linux:** Open a terminal and run `hostname -I` or `ip addr`.
- **在Linux上:** 打开终端并运行 `hostname -I` 或 `ip addr`。

Let's assume your local IP address is `192.168.1.100`.
我们假设您的本地IP地址是 `192.168.1.100`。

## 2. Verify Port Accessibility
## 2. 验证端口可访问性

After starting your server, you can verify that it's listening on the correct network interface and port.
启动服务器后，您可以验证它是否正在侦听正确的网络接口和端口。

- **On Windows:** Open Command Prompt or PowerShell and run `netstat -ano | findstr ":8000"`. Replace `8000` with the port your application is using.
- **在Windows上:** 打开命令提示符或PowerShell并运行 `netstat -ano | findstr ":8000"`。 将 `8000` 替换为您的应用程序正在使用的端口。

- **On macOS and Linux:** Open a terminal and run `sudo lsof -i :8000` or `netstat -anp | grep ":8000"`.
- **在macOS和Linux上:** 打开终端并运行 `sudo lsof -i :8000` 或 `netstat -anp | grep ":8000"`。

If your server is correctly configured to be accessible from your LAN, you should see an entry with `0.0.0.0:8000` or `*:8000`. This means the service is listening on all available network interfaces. If you see `127.0.0.1:8000` or `localhost:8000`, it's only accessible from your own computer.
如果您的服务器已正确配置为可从您的局域网访问，您应该会看到一个带有 `0.0.0.0:8000` 或 `*:8000` 的条目。 这意味着该服务正在侦听所有可用的网络接口。 如果您看到 `127.0.0.1:8000` 或 `localhost:8000`，则只能从您自己的计算机访问它。

## 3. Configure Windows Firewall
## 3. 配置Windows防火墙

You need to allow incoming connections to the ports your applications are running on.
您需要允许到您的应用程序正在运行的端口的传入连接。

1.  Open "Windows Defender Firewall with Advanced Security".
1.  打开“高级安全Windows Defender防火墙”。

2.  Go to "Inbound Rules" and click "New Rule...".
2.  转到“入站规则”并单击“新建规则...”。

3.  Select "Port" and click "Next".
3.  选择“端口”并单击“下一步”。

4.  Select "TCP" and "Specific local ports". Enter the ports you need to open, separated by commas (e.g., `8000, 5173`).
4.  选择“TCP”和“特定本地端口”。 输入您需要打开的端口，以逗号分隔（例如，`8000, 5173`）。

5.  Click "Next", select "Allow the connection", and click "Next".
5.  单击“下一步”，选择“允许连接”，然后单击“下一步”。

6.  Choose the network profiles for which this rule should apply (Domain, Private, Public). For a home network, "Private" should be sufficient.
6.  选择此规则应适用于的网络配置文件（域、专用、公用）。 对于家庭网络，“专用”就足够了。

7.  Give the rule a name (e.g., "Local Development Servers") and click "Finish".
7.  为规则命名（例如，“本地开发服务器”）并单击“完成”。

## 4. Configure Your Django Application
## 4. 配置您的Django应用程序

### 4.1. Update `ALLOWED_HOSTS`
### 4.1. 更新 `ALLOWED_HOSTS`

In your Django project's `settings.py` file, you need to add your computer's local IP address to the `ALLOWED_HOSTS` list. You can also use a wildcard `*` to allow all hosts, but be aware that this is less secure.
在Django项目的 `settings.py` 文件中，您需要将计算机的本地IP地址添加到 `ALLOWED_HOSTS` 列表中。 您也可以使用通配符 `*` 来允许所有主机，但请注意，这样不太安全。

```python
# settings.py

# It's recommended to add your specific IP address.
# 建议添加您的特定IP地址。
ALLOWED_HOSTS = ['192.168.1.100', 'localhost', '127.0.0.1']

# Or, for development purposes, you can allow all hosts (less secure)
# 或者，出于开发目的，您可以允许所有主机（不太安全）
# ALLOWED_HOSTS = ['*']
```

### 4.2. Run the Development Server
### 4.2. 运行开发服务器

Start the Django development server and bind it to `0.0.0.0` to make it accessible from other devices on the network.
启动Django开发服务器并将其绑定到 `0.0.0.0` 以使其可从网络上的其他设备访问。

```bash
python manage.py runserver 0.0.0.0:8000
```

Now, you should be able to access your Django application from another device on the same LAN at `http://192.168.1.100:8000`.
现在，您应该可以从同一局域网上的另一台设备通过 `http://192.168.1.100:8000` 访问您的Django应用程序。

## 5. Configure Your Vue.js (Vite) Application
## 5. 配置您的Vue.js (Vite) 应用程序

### 5.1. Run the Development Server
### 5.1. 运行开发服务器

To make the Vite development server accessible from the LAN, use the `--host` flag when starting it.
要使Vite开发服务器可从局域网访问，请在启动时使用 `--host` 标志。

```bash
npm run dev -- --host
```

This will bind the server to `0.0.0.0`.
这会将服务器绑定到 `0.0.0.0`。

### 5.2. Alternative: Configure `vite.config.js`
### 5.2. 替代方法: 配置 `vite.config.js`

You can also configure the server host directly in your `vite.config.js` file. This is a more permanent solution.
您也可以直接在 `vite.config.js` 文件中配置服务器主机。 这是一个更永久的解决方案。

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  server: {
    host: '0.0.0.0'
  }
})
```

Now, you can simply run `npm run dev` and it will be accessible from the LAN.
现在，您只需运行 `npm run dev` 即可从局域网访问。

### 5.3. Handling API Requests to Your Django Backend
### 5.3. 处理对Django后端的API请求

If your Vue.js application makes requests to your Django backend, you'll likely need to update the base URL for your API calls from `http://localhost:8000` or `http://127.0.0.1:8000` to your local IP address, like `http://192.168.1.100:8000`.
如果您的Vue.js应用程序向您的Django后端发出请求，您可能需要将API调用的基本URL从 `http://localhost:8000` 或 `http://127.0.0.1:8000` 更新为您的本地IP地址，例如 `http://192.168.1.100:8000`。

A better solution for development is to use Vite's proxy feature. This allows you to keep your API requests in your frontend code pointing to a relative path, and Vite will proxy them to your Django backend.
一个更好的开发解决方案是使用Vite的代理功能。 这允许您将前端代码中的API请求指向相对路径，Vite会将其代理到您的Django后端。

In `vite.config.js`:
在 `vite.config.js` 中:

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  server: {
    host: '0.0.0.0',
    proxy: {
      // string shorthand
      // 字符串速记
      // '/api': 'http://localhost:8000',
      // with options
      // 带选项
      '/api': {
        target: 'http://127.0.0.1:8000',
        changeOrigin: true,
        // If you are using Django Rest Framework and get CSRF errors,
        // you might need to rewrite the origin header.
        // 如果您正在使用Django Rest Framework并收到CSRF错误，
        // 您可能需要重写origin标头。
        // configure: (proxy, options) => {
        //   proxy.on('proxyReq', (proxyReq, req, res) => {
        //     proxyReq.setHeader('Origin', 'http://127.0.0.1:8000');
        //   });
        // }
      }
    }
  }
})
```

With this configuration, a request to `/api/some-endpoint` in your Vue app will be forwarded to `http://127.0.0.1:8000/api/some-endpoint`.
通过此配置，对您的Vue应用中 `/api/some-endpoint` 的请求将被转发到 `http://127.0.0.1:8000/api/some-endpoint`。

Now, you should be able to access your Vue.js application from another device on the same LAN at `http://192.168.1.100:5173` (or whatever port Vite is using).
现在，您应该可以从同一局域网上的另一台设备通过 `http://192.168.1.100:5173` (或Vite正在使用的任何端口) 访问您的Vue.js应用程序。
