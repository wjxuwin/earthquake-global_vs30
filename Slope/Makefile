#
# Makefile to generate a global slope-based Vs30 file
#
include ../Constants.mk

.PHONY: all clean veryclean

OLD_GRD = $(wildcard ../global_vs30_*.grd)

ifneq ($(strip $(OLD_GRD)),)
all : 
	$(warning Skipping Slope processing: The global Vs30 directory\
                $Nalready contains a global grid file with one or more regional stamps.\
                $NIf you want to start fresh, remove '$(OLD_GRD)')
else
all : ../global_vs30.grd
endif

../global_vs30.grd : global_vs30.grd
	cp global_vs30.grd ..

global_vs30.grd : global_grad.grd global_landmask.grd cratons_smooth.grd \
                  ../src/grad2vs30
	../src/grad2vs30 gradient_file=global_grad.grd craton_file=cratons_smooth.grd landmask_file=global_landmask.grd output_file=global_vs30.grd water=$(WATER)

clean :
	$(RM) cratons.bil cratons.hdr cratons.grd cratons_smooth.grd \
              global_landmask.grd elev/gmted_global.bil gmted_global.grd \
              global_grad.grd cratons_pixel.grd

veryclean : clean
	$(RM) -rf elev
	$(RM) -rf cratons
	$(RM) global_vs30.grd

#######################
#
# Create the slope file from the DEM (the -G option isn't necessary on 
# newer versions of GMT):
#
global_grad.grd : gmted_global.grd
	grdgradient gmted_global.grd -Lg -M -D -Sglobal_grad.grd -Gjunk.grd -V
	$(RM) junk.grd

#
# Convert .bil to pixel registered .grd, then convert it to gridline 
# node registration:
#
gmted_global.grd : elev/gmted_global.bil
	xyz2grd -Zh -F -R$(GLOBAL_REGION) -I$(RES)c elev/gmted_global.bil \
                -Ggmted_global_pixel.grd
	grdsample gmted_global_pixel.grd -Ggmted_global.grd -T -fg
	$(RM) gmted_global_pixel.grd

#
# Convert the DEM to .bil:
#
elev/gmted_global.bil : elev/md$(GRES)_grd
	$(GDAL_PATH)/gdal_translate -of EHdr elev/md$(GRES)_grd elev/gmted_global.bil

#
# Unzip the DEM; we have to touch the dir and files because they
# are older than the archive that contains them, forcing make
# to always repeat this (and all following) step(s)
#
elev/md$(GRES)_grd : elev/md$(GRES)_grd.zip
	(cd elev && unzip md$(GRES)_grd.zip && touch -c md$(GRES)_grd && \
         touch md$(GRES)_grd/* )

#
# Get the DEM:
#
elev/md$(GRES)_grd.zip :
	mkdir -p elev;
	$(CURL) http://edcintl.cr.usgs.gov/downloads/sciweb1/shared/topo/downloads/GMTED/Grid_ZipFiles/md$(GRES)_grd.zip > elev/md$(GRES)_grd.zip 

#
# Create the landmask:
#
global_landmask.grd : 
	grdlandmask -V -R$(GLOBAL_REGION) -I$(RES)c -Gglobal_landmask.grd -Df

#
# Smooth the craton file
#
cratons_smooth.grd : cratons.grd ../src/smooth
	../src/smooth infile=cratons.grd fx=$(GLOBE_FX) fy=$(GLOBE_FY) outfile=cratons_smooth.grd


#
# Convert the craton.bil to a pixel registered .grd file, then convert that to gridline 
# node registration; use -Qn to keep everything at 0 and 1:
#
cratons.grd : cratons.bil
	xyz2grd -R$(GLOBAL_REGION) -I$(RES)c -Zc -F cratons.bil -Gcratons_pixel.grd
	grdsample cratons_pixel.grd -Gcratons.grd -T -fg -Qn
	$(RM) cratons_pixel.grd

#
# Create the craton file from the shape files:
#
cratons.bil : cratons/cratons.shp
	$(GDAL_PATH)/gdal_rasterize -burn 1 -of EHdr -init 0 -te $(GXMIN) $(GYMIN) $(GXMAX) $(GYMAX) -tr $(RES_DD) $(RES_DD) -ot Byte cratons/cratons.shp cratons.bil

cratons/cratons.shp : cratons.tgz
	tar xvmf cratons.tgz

cratons.tgz :
	echo "File cratons.tgz must be supplied -- cannot make"


#
# Make the C programs
#
../src/smooth :
	$(MAKE) -C ../src smooth

../src/grad2vs30 :
	$(MAKE) -C ../src grad2vs30

######################################################################
# Make some plots
######################################################################
plot_cratons : cratons.png
	
plot_cratons_smooth : cratons_smooth.png

plot_global_topo : global_topo.png

plot_global_vs30 : global_vs30.png
	
cratons.png : cratons.grd
	grdimage cratons.grd -JM18 -R-180/180/-56/72 -C$(WEIGHTS_CPT) -Ba24d/a12eWSen -K > cratons.ps
	pscoast -JM18 -R-180/180/-56/72 -Di -N1 -W -S128/128/255 -A1000/0/2 -O -K >> cratons.ps
#	psscale -D20/4.3/9/0.5 -L -C$(WEIGHTS_CPT) -O >> cratons.ps
	convert -rotate 90 -density 300x300 -crop 2400x1400+100+950 cratons.ps cratons.png

cratons_smooth.png : cratons_smooth.grd
	grdimage cratons_smooth.grd -JM18 -R-130/-48/12/50 -C$(WEIGHTS_CPT) -Ba8d/a4eWSen -K > cratons_smooth.ps
	pscoast -JM18 -R-130/-48/12/50 -Di -N1 -N2 -W -S128/128/255 -A500/0/2 -O -K >> cratons_smooth.ps
	psscale -D19/4.3/9/0.5 -L -C$(WEIGHTS_CPT) -O >> cratons_smooth.ps
	convert -rotate 90 -density 300x300 -crop 2700x1400+100+950 cratons_smooth.ps cratons_smooth.png
	
global_topo.png : gmted_global.grd
	grdimage gmted_global.grd -JM20 -R-180/180/-56/72 -C../Misc/tan.cpt -Ba24d/a12eWSen -K > global_topo.ps
	pscoast -JM20 -R-180/180/-56/72 -Di -N1 -W -S128/128/255 -A1000/0/2 -O -K >> global_topo.ps
#	psscale -D21/4.3/9/0.5 -L -C../Misc/tan.cpt -O >> global_topo.ps
	convert -rotate 90 -density 300x300 -crop 2700x1400+100+950 global_topo.ps global_topo.png

global_vs30.png : global_vs30.grd
	grdimage global_vs30.grd -JM20 -R-180/180/-56/72 -C../Misc/global.cpt -Ba24d/a12eWSen -K > global_vs30.ps
	pscoast -JM20 -R-180/180/-56/72 -Di -N1 -W -S128/128/255 -A1000/0/2 -O -K >> global_vs30.ps
	psscale -D21/4.3/9/0.5 -L -C../Misc/global.cpt -O >> global_vs30.ps
	convert -rotate 90 -density 300x300 -crop 3000x1400+100+950 global_vs30.ps global_vs30.png
