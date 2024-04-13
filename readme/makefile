# 定义编译器
FC = gfortran

# 定义编译选项
FFLAGS = -O3

# 定义源文件列表
SRCS = main_int.f90 slope.f90 bee8.f90 checon.f90 deemat.f90 dismsh.f90 emb_2d_bc.f90 emb_2d_geom.f90 fkdiag.f90 formm.f90 fsparv.f90 g_prop.f90 getname.f90 invar.f90 meshWT.f90 mocouf.f90 mocouq.f90 num_to_g.f90 sample.f90 shape_fun.f90 spabac.f90 sparin.f90 Vanapalli.f90 vecmsh.f90 water_loads.f90 WTpoints.f90

# 定义目标文件列表
OBJS = $(SRCS:.f90=.o)

# 定义可执行文件名
EXE = slope_pore_stress

# 默认目标
all: $(EXE)

# 链接目标文件生成可执行文件
$(EXE): $(OBJS)
	$(FC) $(FFLAGS) -o $@ $^

# 编译源文件生成目标文件
%.o: %.f90
	$(FC) $(FFLAGS) -c $<

# 清理目标
clean:
	rm -f $(OBJS)
# $(EXE)
