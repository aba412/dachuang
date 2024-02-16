# *概述*

example_spatial_02的代码主要展示了如何使用 ICASAR 处理 LiCSBAS 输出的数据.包括载入 LiCSBAS 的 .h5 文件，以及使用 ICASAR 进行独立成分分析（ICA）来提取地表变形信号。具体包括了对增量干涉图和累积干涉图进行不同的处理方式，以及展示了如何可视化拟合干涉图的结果。  

### 加载和准备 LiCSBAS 输出的数据，并将所需传入[字典](dictionary)spatial_data中

```
LiCSBAS_out_folder_campi_flegrei = Path('./campi_flegrei_LiCSBAS_example_data/')  #创建一个文件夹路径对象
print(f"Opening the LiCSBAS .h5 file...", end = '')
 
displacement_r2, tbaseline_info, ref_xy = LiCSBAS_to_ICASAR(LiCSBAS_out_folder_campi_flegrei, figures=True)
#上面那行代码是在调用 LiCSBAS_to_ICASAR 函数来处理 LiCSBAS 输出的数据，
#并将返回值分别赋值给displacement_r2, tbaseline_info, ref_xy       
print(f"Done.  ")

spatial_data = {'ifgs_dc'        : displacement_r2['incremental'], #增量变形数据
                'mask'           : displacement_r2['mask'],        #遮罩数据，即掩膜
                'ifg_dates_dc'   : tbaseline_info['ifg_dates'],    #干涉图日期信息          
                'dem'            : displacement_r2['dem'],         #数字高程模型数据    
                'lons'           : displacement_r2['lons'],        #经度数据
                'lats'           : displacement_r2['lats']}        #纬度数据
```
##### **示例一**
```
#%% Example 1: sICA after creating all interferograms  

ICASAR_settings = {"n_comp" : 5,                                         
                    "bootstrapping_param" : (200, 0),                    
                    "hdbscan_param" : (35, 10),                        
                    "tsne_param" : (30, 12),                             
                    "ica_param" : (1e-2, 150),                            
                    "hdbscan_param" : (100,10),                          
                    "out_folder" : Path('example_spatial_02_outputs_sICA'),   
                    "load_fastICA_results" : True,                      
                    "ifgs_format" : 'all',                    
                    'sica_tica'         : 'sica',                        
                    'max_n_all_ifgs' :  1000,
                    'label_sources'         : True,                     
                    "figures" : "png+window"}  
                
S_ica, A_ica, x_train_residual_ts, Iq, n_clusters, S_all_info, phUnw_mean, sources_labels  = ICASAR(spatial_data = spatial_data, **ICASAR_settings) 

visualise_ICASAR_inversion(spatial_data['ifgs_dc'], S_ica, A_ica, displacement_r2['mask'], n_data = 10)
```
该示例使用所有干涉图像进行sICA。设置了ICASAR的参数，调用ICASAR函数并返回处理后的结果。最后调用visualise_ICASAR_inversion函数，将干涉图像、S_ica、A_ica和掩膜传入，以可视化展示如何使用学习到的组件（ICs）来拟合干涉图像。
##### **示例二**
```
#%% Example 2: sICA with only incremental interferograms

ICASAR_settings = {"n_comp" : 5,                                         
                    "bootstrapping_param" : (200, 0),                    
                    "hdbscan_param" : (35, 10),                        
                    "tsne_param" : (30, 12),                             
                    "ica_param" : (1e-2, 150),                            
                    "hdbscan_param" : (100,10),                          
                    "out_folder" : Path('example_spatial_02_outputs_sICA_incremental'),   
                    "load_fastICA_results" : True,                      
                    "ifgs_format" : 'inc',                      
                    'max_n_all_ifgs' :  1000,
                    'sica_tica'         : 'sica',                        
                    'label_sources'         : True,                     
                    "figures" : "png+window"}                            

S_ica, A_ica, x_train_residual_ts, Iq, n_clusters, S_all_info, phUnw_mean, sources_labels = ICASAR(spatial_data = spatial_data, **ICASAR_settings) 
```
该示例使用增量干涉图像进行sICA。同样地，设置ICASAR的参数，调用ICASAR函数并返回处理后的结果。
##### **示例三**
```
#%% Example 3: tICA given the incremental interferograms, but this will calculate the cumulative interferograms.

ICASAR_settings = {"n_comp" : 5,                                         
                   "bootstrapping_param" : (200, 0),                    
                   "hdbscan_param" : (35, 10),                        
                   "tsne_param" : (30, 12),                             
                   "ica_param" : (1e-2, 150),                            
                   "hdbscan_param" : (100,10),                          
                   "out_folder" : Path('example_spatial_02_outputs_tICA_incremental'),   
                   "load_fastICA_results" : True,                      
                   'max_n_all_ifgs' :  1000,
                   'sica_tica'         : 'tica',                        
                    'label_sources'         : True,                     
                   "figures" : "png+window"}                            

S_ica, A_ica, x_train_residual_ts, Iq, n_clusters, S_all_info, phUnw_mean, sources_labels = ICASAR(spatial_data = spatial_data, **ICASAR_settings) 
```
该示例使用增量干涉图像进行tICA。同样地，设置ICASAR的参数，调用ICASAR函数并返回处理后的结果。

### 相关名词或参量名或涉及的算法（不分先后）
* 名词
  * 字典<a name="dictionary"></a>
  * 掩膜
* 参量名
* 涉及的算法
