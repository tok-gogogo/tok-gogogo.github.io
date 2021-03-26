---
layout: mypost
title: android性能1-内存、CPU获取
categories: [Android]
---
### **内存**
通过读取文件"/proc/meminfo"的信息能够获取手机Memory的总量，而通过ActivityManager.getMemoryInfo(ActivityManager.MemoryInfo)方法可以获取当前的可用Memory量。
#### **获取总内存，剩余内存，pid内存**
```
public long getTotalMemory() {
		String memInfoPath = "/proc/meminfo";
		String readTemp = "";
		String memTotal = "";
		long memory = 0;
		try {
			FileReader fr = new FileReader(memInfoPath);
			BufferedReader localBufferedReader = new BufferedReader(fr, 8192);
			while ((readTemp = localBufferedReader.readLine()) != null) {
				if (readTemp.contains("MemTotal")) {
					String[] total = readTemp.split(":");
					memTotal = total[1].trim();
				}
			}
			localBufferedReader.close();
			String[] memKb = memTotal.split(" ");
			memTotal = memKb[0].trim();
			Log.d(LOG_TAG, "memTotal: " + memTotal);
			memory = Long.parseLong(memTotal);
		} catch (IOException e) {
			Log.e(LOG_TAG, "IOException: " + e.getMessage());
		}
		return memory;
	}
public long getFreeMemorySize(Context context) {
		ActivityManager.MemoryInfo outInfo = new ActivityManager.MemoryInfo();
		ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
		am.getMemoryInfo(outInfo);
		long avaliMem = outInfo.availMem;
		return avaliMem / 1024;
	}
public int getPidMemorySize(int pid, Context context) {
		ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
		int[] myMempid = new int[] { pid };
		Debug.MemoryInfo[] memoryInfo = am.getProcessMemoryInfo(myMempid);
		memoryInfo[0].getTotalSharedDirty();
		int memSize = memoryInfo[0].getTotalPss();
		return memSize;
}
```
### **CPU**
同样的，通过通过读取文件"/proc/cpuinfo"和"/proc/pid/stat"的信息能够获取手机Cpu的总量,pid应用占用率
#### **获取总cpu和pid应用cpu**
```
public void readCpu(int pid) {
		String processPid = Integer.toString(pid);
		String cpuStatPath = "/proc/" + processPid + "/stat";
		try {
			// monitor cpu stat of certain process
				RandomAccessFile processCpuInfo = new RandomAccessFile(cpuStatPath , "r");
			String line = "";
			StringBuffer stringBuffer = new StringBuffer();
			stringBuffer.setLength(0);
			while ((line = processCpuInfo.readLine()) != null) {
				stringBuffer.append(line + "\n");
			}
			String[] tok = stringBuffer.toString().split(" ");
			processCpu = Long.parseLong(tok[13]) + Long.parseLong(tok[14]);
			processCpuInfo.close();
		} catch (FileNotFoundException e) {
			Log.w(LOG_TAG, "FileNotFoundException: " + e.getMessage());
		} catch (IOException e) {
			e.printStackTrace();
		}
		try {
			// monitor total and idle cpu stat of certain process
			RandomAccessFile cpuInfo = new RandomAccessFile("/proc/cpuinfo", "r");
			String line = "";
			while ((null != (line = cpuInfo.readLine())) && line.startsWith("cpu")) {
				String[] toks = line.split("\\s+");
				idleCpu.add(Long.parseLong(toks[4]));
				totalCpu.add(Long.parseLong(toks[1]) + Long.parseLong(toks[2]) + Long.parseLong(toks[3]) + Long.parseLong(toks[4])
						+ Long.parseLong(toks[6]) + Long.parseLong(toks[5]) + Long.parseLong(toks[7]));
			}
			cpuInfo.close();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
}
```